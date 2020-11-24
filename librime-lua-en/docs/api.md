# Module

> All librime-lua module should export as a table like `{init=function, func=function, fini=function}`

**Tips:** *in this document parameters were presented as `function(parameter : type)` or `function(type)`*

## Example
**example.schema.yaml**

```YAML
engine:	# binding lua_function to rime components
  translators:
    - lua_translator@lua_tran	# lua_tran.func(input,Segment,env)
  filters:
    - lua_filter@lua_filt # lua_filt.func(input)
  processor:
    - lua_processor@lua_proc # lua_proc.func(KeyEvent,env)
```

**rime.lua**

```lua
lua_tran = require("aTranslator")
lua_filt = require("aFilter")
lua_proc = require("aProcessor")
```

**aTranslator.lua**

```lua

local function main(input,segment,env)
	print(env.hello)
	-- helloRime
end

local function destory()
	print("Destroied!")
end

local function prepare(env)
	env.hello = "helloRime"
end

return {init = prepare,func = main,fini = destory}
```

**aFilter.lua**

```lua
local function filter(input)
   local l = {}
   for cand in input:iter() do
      if (utf8.len(cand.text) == 1) then
	 yield(cand)
      else
	 table.insert(l, cand)
      end
   end
   for i, cand in ipairs(l) do
      yield(cand)
   end
end

return filter
-- or 
-- return {init = nil,func = filter, fini = nil}
```

**aProcessor**

```
local function filter(key,env)
	print(key.keycode)
end

local function init(env)
	print(env.engine.schema.schema_id)
end

return {init = init, func = filter, fini = nil}
```

## init

> `init(env)` would be called when Initializing. 

> `env` is the global variable of librime-Lua containing the librime environment, which has the following properties.

> `env.engine` type of *engine* and `env.name_space` type of *string*

> You could bind variables to env and use it in your module.

```lua
-- in test.lua

function ModuleInit(env)
	env.memory = Memory(env.engine,env.engine.schema)
	env.pyrdb = ReverseDb("build/terra_pinyin.reverse.bin")
end

return {init = ModuleInit}
```

## func

> `func` would be called when rime is attempting to call the module.

> Which parameters would be passed depending on which rime component bind with the Lua module.

> If Module bind with Translator. `func(input, Segment, env)` Would be called by `Translator::Query`. tips: `input` is type of `string`

> If Module bind with Filter. `func(Translation, env)` would be called `Filter::apply`

> If Module bind with Processor. `func(KeyEvent, env)` would be called by `Processor::ProcessKeyEvent`


## fini

> When LuaTranslator, LuaFilter, LuaProcess being destoried `fini` would be called 

## librime-lua built-in component

### DictEntry
> #### Functions
> > ##### DictEntry()
> #### Getters
> > ##### text
> > ##### comment
> > ##### preedit
> > ##### weight
> > ##### commit_count
> > ##### custom_code
> > ##### remaining_code_length
> > ##### code
> #### Setters
> > ##### text
> > ##### comment
> > ##### preedit
> > ##### weight
> > ##### commit_count
> > ##### custom_code
> > ##### remaining_code_length
> > ##### code
### Code
> #### Functions
> > ##### Code()
> #### Methods
> > ##### push
> > ##### print
### CommitEntry
> #### Functions
> > ##### CommitEntry()
> #### Methods
> > ##### get()
### Memory

> #### Introduction

> > A compoent that could access to `Dictionary` and `UserDictionary`

> #### Functions

> > ##### Memory(Engine, Schema)

> > > Returns the Instance of *Memory*

> > > Instance a `Memory` Object with given `Engine` and `Scehma`

> > ##### CustomMemory(Engine, schema_id="", namespace="translator") 

> > > Returns the Instance of *Memory*

> > > Instance a `Memory` Object with given `Engine` (running Engine), a `scheme_id`, and `namespace`

> > > `scheme_id` were defined on input method schema

*cangjie5.schema.yaml*

```yaml
schema:
  schema_id: cangjie5 # this one
  name: 倉頡五代
  version: "1.1.test"
```

> > > `namespace` is used to collect yaml tag of `namespace/dictionary` `namespace/enable_user_dict` and `namespace/db_class` in schema.

*cangjie5.schema.yaml*

```yaml
translator:
  dictionary: cangjie5 #this one!
  # enable_user_dict: true #this one
  # db_class: tabledb #this one
  enable_charset_filter: true
  enable_sentence: true
  enable_encoder: true
  encode_commit_history: true
  max_phrase_length: 5
  preedit_format:
    - 'xform/^([a-z]*)$/$1\t（\U$1\E）/'
    - "xlit|ABCDEFGHIJKLMNOPQRSTUVWXYZ|日月金木水火土竹戈十大中一弓人心手口尸廿山女田難卜符|"
  comment_format:
    - "xlit|abcdefghijklmnopqrstuvwxyz~|日月金木水火土竹戈十大中一弓人心手口尸廿山女田難卜符～|"
  disable_user_dict_for_patterns:
    - "^z.*$"
    - "^yyy.*$"
```

> #### Methods

> > ##### dictLookup(input : string, isExpand : boolean)

> > > Returns *true* if Lookup success.

> > > The results are stored in `DicionaryIterator` inside `Memory`.

> > > If `isExpand` were true, it would expand your `input` as a prefix. For example, the input is "hq", all results with the prefix "hq" in code  would be output.

> > ##### userLookup(input : string, isExpand : boolean)

> > > Returns *true* if Lookup success.

> > > The results are stored in `UserDicionaryIterator` inside `Memory`.

> > > If `isExpand` were true, it would expand your `input` as a prefix. For example, the input is "hq", all results with the prefix "hq" in code  would be output.

> > ##### iter_dict()

> > > Returns a Lua Iterator Of `Dictionary` Lookup Results which iterates a dictEntry.

> > ##### iter_user()

> > > Returns a Lua Iterator Of `UserDictionary` Lookup Results which iterates a dictEntry.

> > ##### memorize(function(CommitEntry))

> > > To bind a lua_Function to `Memory::memorize` The Function would be called after commit, passing `CommitEntry` (contains committed `dictEntry`) to it.

> > > You could use CommitEntry to manage User_Dict

> > ##### updateUserdict(DictEntry, commits : number, new_entry_prefix : string)

> > > Update `UserDictionary` with specific `DictEntry`

> > > If `commits` < 0 this entry would be marked as deleted.

> > > If `commits` > 0 then increasing the frequency of this entry committed.

> > > `new_entry_prefix` would become the code prefix of this entry  

### Segment

> #### Functions
> > ##### Segment(start_pos : number, end_pos : number)
> > > Returns Instance of Segment
> #### Methods
> > ##### clear()
> > > clean tags, reset menu,set Selected_index to 0,Clear prompt


> > ##### close()
> > > auto select Candidate then split into selected segment and  unselcted segment

> > ##### reopen(caret_pos : number)
```C++
bool Segment::Reopen(size_t caret_pos) {
  if (status < kSelected) {
    return false;
  }
  const size_t original_end_pos = start + length;
  if (original_end_pos == caret_pos) {
    // reuse previous candidates and keep selection
    if (end < original_end_pos) {
      // restore partial-selected segment
      end = original_end_pos;
      tags.erase(kPartialSelectionTag);
    }
    status = kGuess;
  }
  else {
    status = kVoid;
  }
  return true;
}
```
> > ##### has_tag(tag : string)
> > > Returns *if* the tag has been found.
> > ##### get_candidate_at(index : number)
> > > Returns `Candidate` of specific index.
> > ##### get_selected_candidate()
> > > Returns `Candidte` of selected candidte.

> #### Getters
> > ##### status : string
> > > Returns `"kVoid"` `"kVGuess"` `"kselected"` or `"kConfirmed"`
> > ##### start : number
> > ##### \_end : number
> > ##### length : number
> > > Returns Segment's length
> > ##### tags : table
> > > Returns a set of tags
> > ##### menu : Menu
> > ##### selected_index : number
> > ##### prompt : string

> #### Setters
> > ##### status : string
> > ##### start : number
> > ##### \_end : number
> > ##### length : number
> > ##### tags : table
> > ##### menu : Menu
> > ##### selected_index : number
> > ##### prompt : string

### Candidate
> #### Functions
> > ##### Candidate(type : string, start : number, end: number, text : string, comment : string)
> > > Returns Instance of `Candidate`
> #### Methods
> > ##### get_dynamic_type()
> > ##### get_genuine(Candidate)
> > ##### get_genuines(Candidate)
> #### Getters
> > ##### type
> > ##### start
> > ##### \_end
> > ##### quality
> > ##### text
> > ##### comment
> > ##### preedit
> #### Setters
> > ##### type
> > ##### start
> > ##### \_end
> > ##### quality
> > ##### text
> > ##### comment
> > ##### preedit
### Translation
> #### Functions
> > ##### Translation(initFunction : function)
> > > Returns Instance of `Translation`,and call the initFunction.
> #### Methods
> > ##### iter()
> > > Returns lua iterator for `Translation` that iterate `Candidate`
### ReverseDb
> #### Functions
> > ##### ReverseDb(ReverseDb_File_Name : string)
> > > Returns Instance of `ReverseDb`
> #### Methods
> > ##### lookup(key : string)
> > > Returns `string` of reverseDb lookup result

### Segmentation
> #### Methods
> > ##### empty
> > ##### back
> > ##### pop_back
> > ##### reset_length
> > ##### add_segment
> > ##### forward
> > ##### trim
> > ##### has_finished_segmentation
> > ##### get_current_start_position
> > ##### get_current_end_position
> > ##### get_current_segment_length
> > ##### get_confirmed_position
> #### Getters
> > ##### input : string
> #### Setters
> > ##### input : string
### Menu
> #### Functions
> > ##### add_translation
> > ##### prepare
> > ##### get_candidate_at
> > ##### candidate_count
> > ##### empty
### KeyEvent
> #### Methods
> > ##### shift
> > ##### ctrl
> > ##### alt
> > ##### caps
> > ##### super
> > ##### release
> > ##### repr
> > ##### eq
> > ##### lt
> #### Getters
> > ##### keycode
> > ##### modifier
### Engine
> #### Methods
> > ##### commit_text
> #### Getters
> > ##### schema
> > ##### context
> > ##### active_engine
> #### Setters
> > ##### active_engine
### Context
> #### Methods
> > ##### commit
> > ##### get_commit_text
> > ##### get_script_text
> > ##### get_preedit
> > ##### is_composing
> > ##### has_menu
> > ##### get_selected_candidate
> > ##### push_input
> > ##### pop_input
> > ##### delete_input
> > ##### clear
> > ##### select
> > ##### confirm_current_selection
> > ##### delete_current_selection
> > ##### confirm_previous_selection
> > ##### reopen_previous_selection
> > ##### clear_previous_segment
> > ##### reopen_previous_segment
> > ##### clear_non_confirmed_composition
> > ##### refresh_non_confirmed_composition
> > ##### set_option
> > ##### get_option
> > ##### set_property
> > ##### get_property
> > ##### clear_transient_options

> #### Getters
> > ##### composition
> > ##### input
> > ##### caret_pos
> > ##### commit_notifier
> > ##### select_notifier
> > ##### update_notifier
> > ##### delete_notifier
> > ##### option_update_notifier
> > ##### property_update_notifier
> > ##### unhandled_key_notifier
> #### Setters
> > ##### composition
> > ##### input
> > ##### caret_pos
### Preedit
> #### Getters
> > ##### text
> > ##### caret_pos
> > ##### sel_start
> > ##### sel_end
> #### Setters
> > ##### text
> > ##### caret_pos
> > ##### sel_start
> > ##### sel_end
### Composition
> #### Methods
> > ##### empty
> > ##### back
> > ##### pop_back
> > ##### push_back
> > ##### has_finished_composition
> > ##### get_prompt
### Schema
> #### Functions
> > ##### Schema(schema_id : string)
> #### Getters
> > ##### schema_id
> > ##### schema_name
> > ##### config
> > ##### page_size
> > ##### select_keys
> #### Setters
> > ##### config
> > ##### select_keys
### Config
> #### Methods
> > ##### load_from_file
> > ##### save_to_file
> > ##### is_null
> > ##### is_value
> > ##### is_list
> > ##### is_map
> > ##### get_bool
> > ##### get_int
> > ##### get_double
> > ##### get_string
> > ##### get_list_size
> > ##### set_bool
> > ##### set_int
> > ##### set_double
> > ##### set_string
### Connection
> #### Methods
> > ##### disconnect
### Notifier
> #### Methods
> > ##### connect
### OptionUpdateNotifier
> #### Methods
> > ##### connect
### PropertyUpdateNotifier
> #### Methods
> > ##### connect
### KeyEventNotifer
> #### Methods
> > ##### connect
### Log
> #### Functions
> > ##### info
> > ##### warning
> > ##### error