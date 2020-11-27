# Module

> All librime-lua module should export as a table like `{init=function, func=function, fini=function}`

**Tips:** *in this document parameters were presented as `function(parameter : type)` or `function(type)`*
*function() : type represented return type*
*Pasted code is code of C++ source I don't understand*

> You can call Functions of Component globally like `Functions()`

> Call Methods of Component like `ComponentObject:method()`

> Getters means properties can be accessed like `Component.property`

> Setters means properties can be access like `Component.property = value`

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
> > ##### DictEntry() : DictEntry
> > > Instance a empty `DictEntry`
> #### Getters
> > ##### text : string
> > ##### comment : string
> > > if use expand dictionary lookup, it would write raminging code to comment
> > ##### preedit : string
> > ##### weight : number 
> > ##### commit_count : number
> > ##### custom_code : string
> > ##### remaining_code_length : number
> > ##### code : Code
> #### Setters
> > ##### text : string
> > ##### comment : string
> > ##### preedit : string
> > ##### weight : number 
> > ##### commit_count : number
> > ##### custom_code : string
> > ##### remaining_code_length : number
> > ##### code : Code
### Code
> #### Functions
> > ##### Code()
> > > Create Empty `Code`
> #### Methods
> > ##### push(code : number)
> > > push code to `Code`
> > ##### print()
> > > print current code
### CommitEntry
> #### Methods
> > ##### get() : {DictEntry}
> > > Returns array of `DictEntry` in `CommitEntry`
### Memory
  
> #### Introduction

> > A compoent that could access to `Dictionary` and `UserDictionary`

> #### Functions

> > ##### Memory(Engine, Schema) : Memory

> > > Returns the Instance of *Memory*

> > > Instance a `Memory` Object with given `Engine` and `Scehma`

> > ##### CustomMemory(Engine, schema_id="", namespace="translator") : Memory

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

> > ##### dictLookup(input : string, isExpand : boolean) : boolean

> > > Returns *true* if Lookup success.

> > > The results are stored in `DicionaryIterator` inside `Memory`.

> > > If `isExpand` were true, it would expand your `input` as a prefix. For example, the input is "hq", all results with the prefix "hq" in code  would be output.

> > ##### userLookup(input : string, isExpand : boolean) : boolean

> > > Returns *true* if Lookup success.

> > > The results are stored in `UserDicionaryIterator` inside `Memory`.

> > > If `isExpand` were true, it would expand your `input` as a prefix. For example, the input is "hq", all results with the prefix "hq" in code  would be output.

> > ##### iter_dict()

> > > Returns a Lua Iterator Of `Dictionary` Lookup Results which iterates a dictEntry.
```lua
for entry in mem:iter_dict()
 do
 print(entry.text)
end
```
> > ##### iter_user() 

> > > Returns a Lua Iterator Of `UserDictionary` Lookup Results which iterates a dictEntry.

```lua
for entry in mem:iter_user()
 do
 print(entry.text)
end
```

> > ##### memorize(function(CommitEntry))

> > > To bind a lua_Function to `Memory::memorize` The Function would be called after commit, passing `CommitEntry` (contains committed `dictEntry`) to it.

> > > You could use CommitEntry to manage User_Dict

> > ##### updateUserdict(DictEntry, commits : number, new_entry_prefix : string)

> > > Update `UserDictionary` with specific `DictEntry`

> > > If `commits` < 0 this entry would be marked as deleted.

> > > If `commits` > 0 then increasing the frequency of this entry committed.

> > > `new_entry_prefix` would become the code prefix of this entry  

> > ##### decode(Code) : table(string)

> > decode Code to string code

### Segment

> #### Functions
> > ##### Segment(start_pos : number, end_pos : number) : Segment
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
> > ##### has_tag(tag : string) : boolean
> > > Returns *if* the tag has been found.
> > ##### get_candidate_at(index : number) : number
> > > Returns `Candidate` of specific index.
> > ##### get_selected_candidate() : Candidate
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
> > ##### Candidate(type : string, start : number, end: number, text : string, comment : string) : Candidate
> > > Returns Instance of `Candidate`
> #### Methods
> > ##### get_dynamic_type() : string
```c++
string dynamic_type(T &c) {
  if (dynamic_cast<Phrase *>(&c))
    return "Phrase";
  if (dynamic_cast<SimpleCandidate *>(&c))
    return "Simple";
  if (dynamic_cast<ShadowCandidate *>(&c))
    return "Shadow";
  if (dynamic_cast<UniquifiedCandidate *>(&c))
    return "Uniquified";
  return "Other";
}
```
> > ##### get_genuine(Candidate) : Candidate
```c++
an<Candidate>
Candidate::GetGenuineCandidate(const an<Candidate>& cand) {
  auto uniquified = As<UniquifiedCandidate>(cand);
  return UnpackShadowCandidate(uniquified ? uniquified->items().front() : cand);
}
```
> > ##### get_genuines(Candidate) : Candidate
```c++
vector<of<Candidate>>
Candidate::GetGenuineCandidates(const an<Candidate>& cand) {
  vector<of<Candidate>> result;
  if (auto uniquified = As<UniquifiedCandidate>(cand)) {
    for (const auto& item : uniquified->items()) {
      result.push_back(UnpackShadowCandidate(item));
    }
  }
  else {
    result.push_back(UnpackShadowCandidate(cand));
  }
  return result;
}
```
> #### Getters
> > ##### type : string
> > ##### start : number
> > ##### \_end : number
> > ##### quality : number
> > ##### text : string
> > ##### comment : string
> > ##### preedit : string
> #### Setters
> > ##### type : string
> > ##### start : number
> > ##### \_end : number
> > ##### quality : number
> > ##### text : string
> > ##### comment : string
> > ##### preedit : string
### Phrase
> #### Functions
> > ##### Candidate(type : string, start : number, end: number, text : string, comment : string) : Candidate
> > > Returns Instance of `Candidate`
> #### Methods
> > ##### get_dynamic_type() : string
```c++
string dynamic_type(T &c) {
  if (dynamic_cast<Phrase *>(&c))
    return "Phrase";
  if (dynamic_cast<SimpleCandidate *>(&c))
    return "Simple";
  if (dynamic_cast<ShadowCandidate *>(&c))
    return "Shadow";
  if (dynamic_cast<UniquifiedCandidate *>(&c))
    return "Uniquified";
  return "Other";
}
```
> > ##### get_genuine(Candidate) : Candidate
```c++
an<Candidate>
Candidate::GetGenuineCandidate(const an<Candidate>& cand) {
  auto uniquified = As<UniquifiedCandidate>(cand);
  return UnpackShadowCandidate(uniquified ? uniquified->items().front() : cand);
}
```
> > ##### get_genuines(Candidate) : Candidate
```c++
vector<of<Candidate>>
Candidate::GetGenuineCandidates(const an<Candidate>& cand) {
  vector<of<Candidate>> result;
  if (auto uniquified = As<UniquifiedCandidate>(cand)) {
    for (const auto& item : uniquified->items()) {
      result.push_back(UnpackShadowCandidate(item));
    }
  }
  else {
    result.push_back(UnpackShadowCandidate(cand));
  }
  return result;
}
```
> > ##### toCandidate() : Candidate
> #### Getters
> > ##### language : Language 
> > > Not wrap to lua yet
> > ##### type : string
> > ##### start : number
> > ##### \_end : number
> > ##### quality : number
> > ##### text : string
> > ##### comment : string
> > ##### preedit : string
> > ##### weight : number
> > ##### code : Code
> > ##### entry : DictEntry
> #### Setters
> > ##### type : string
> > ##### start : number
> > ##### \_end : number
> > ##### quality : number
> > ##### comment : string
> > ##### preedit : string
> > ##### weight : number

### Translation
> #### Functions
> > ##### Translation(initFunction : function)
> > > Returns Instance of `Translation`,and call the initFunction.
> #### Methods
> > ##### iter()
> > > Returns lua iterator for `Translation` that iterate `Candidate`

```lua
for can in translation:iter()
do 
  print(can.text)
end
```

### ReverseDb
> #### Functions
> > ##### ReverseDb(ReverseDb_File_Name : string) : ReverseDb
> > > Returns Instance of `ReverseDb`
> #### Methods
> > ##### lookup(key : string) : string
> > > Returns `string` of reverseDb lookup result

### Segmentation
> #### Methods
> > ##### empty() : boolean
> > ##### back() : Segment
> > > Returns last element of `Segmentation`
> > ##### pop_back()
> > > Delete the last element of `Segmentation` 
> > ##### reset_length(length : number)
> > > Reset `Segmentation` with given length 
> > ##### add_segment(Segment)
> > > Add given `Segment` at `Segmentation`'s end
> > ##### forward()
```C++
// finalize a round
bool Segmentation::Forward() {
  if (empty() || back().start == back().end)
    return false;
  // initialize an empty segment for the next round
  push_back(Segment(back().end, back().end));
  return true;
}
```
> > ##### trim()
```c++
// remove empty trailing segment
bool Segmentation::Trim() {
  if (!empty() && back().start == back().end) {
    pop_back();
    return true;
  }
  return false;
}
```
> > ##### has_finished_segmentation()
```C++
bool Segmentation::HasFinishedSegmentation() const {
  return (empty() ? 0 : back().end) >= input_.length();
}
```
> > ##### get_current_start_position() : number
```c++
size_t Segmentation::GetCurrentStartPosition() const {
  return empty() ? 0 : back().start;
}
```
> > ##### get_current_end_position() : number
```C++
size_t Segmentation::GetCurrentEndPosition() const {
  return empty() ? 0 : back().end;
}

```
> > ##### get_current_segment_length : number
```c++
size_t Segmentation::GetCurrentSegmentLength() const {
  return empty() ? 0 : (back().end - back().start);
}
```
> > ##### get_confirmed_position : number
```c++
size_t Segmentation::GetConfirmedPosition() const {
  size_t k = 0;
  for (const Segment& seg : *this) {
    if (seg.status >= Segment::kSelected)
      k = seg.end;
  }
  return k;
}
```
> #### Getters
> > ##### input : string
> #### Setters
> > ##### input : string
### Menu
> #### Functions
> > ##### add_translation(Translation)
> > ##### prepare(requested : number) : number
```C++
size_t Menu::Prepare(size_t requested) {
  DLOG(INFO) << "preparing " << requested << " candidates.";
  while (candidates_.size() < requested && !result_->exhausted()) {
    if (auto cand = result_->Peek()) {
      candidates_.push_back(cand);
    }
    result_->Next();
  }
  return candidates_.size();
}
```
> > ##### get_candidate_at(index : number) : Candidate
> > ##### candidate_count() : number
> > ##### empty() : boolean
### KeyEvent
> #### Methods
> > ##### shift : boolean
> > ##### ctrl : boolean
> > ##### alt : boolean
> > ##### caps : boolean
> > ##### super : boolean
> > ##### release : boolean
> > ##### repr : string 
> > ##### eq ： boolean
> > > Returns is euqual to other `KeyEvent`
> > ##### lt : boolean
> > > Returns is less than other `KeyEvent`
> #### Getters
> > ##### keycode : number
> > ##### modifier : number
### Engine
> #### Methods
> > ##### commit_text(text : string)
> > > commit given text
> #### Getters
> > ##### schema : Schema
> > > returns current `Schema`
> > ##### context : Context
> > > returns current `Context`
> > ##### active_engine : Engine
> #### Setters
> > ##### active_engine(Engine)
### Context
> #### Methods
> > ##### commit() : boolean
> > ##### get_commit_text() : string
> > ##### get_script_text() : string
> > ##### get_preedit() : string
> > ##### is_composing() : boolean
> > ##### has_menu() : boolean
> > ##### get_selected_candidate() : Candidate
> > ##### push_input(input : string) : boolean
> > ##### pop_input(len : number) : boolean
> > > delete input of given amount
```C++
bool Context::PopInput(size_t len) {
  if (caret_pos_ < len)
    return false;
  caret_pos_ -= len;  // differ to DeleteInput
  input_.erase(caret_pos_, len);
  update_notifier_(this);
  return true;
}
```
> > ##### delete_input(len : number) : boolean
> > > delete input of given amount
```C++
bool Context::DeleteInput(size_t len) {
  if (caret_pos_ + len > input_.length())
    return false;
  input_.erase(caret_pos_, len);
  update_notifier_(this);
  return true;
}

```
> > ##### clear()
> > ##### select(index : number) : boolean
> > ##### confirm_current_selection() : boolean
> > ##### delete_current_selection() : boolean
> > ##### confirm_previous_selection() : boolean
> > ##### reopen_previous_selection() : boolean
> > ##### clear_previous_segment() : boolean
> > ##### reopen_previous_segment() : boolean
> > ##### clear_non_confirmed_composition() : boolean
> > ##### refresh_non_confirmed_composition() : boolean
> > ##### set_option(name : string, value : boolean)
> > ##### get_option(name : string)
> > ##### set_property(name : string,value : string)
> > ##### get_property(name : string) : string
> > ##### clear_transient_options()
```c++
void Context::ClearTransientOptions() {
  auto opt = options_.lower_bound("_");
  while (opt != options_.end() &&
         !opt->first.empty() && opt->first[0] == '_') {
    options_.erase(opt++);
  }
  auto prop = properties_.lower_bound("_");
  while (prop != properties_.end() &&
         !prop->first.empty() && prop->first[0] == '_') {
    properties_.erase(prop++);
  }
}
```

> #### Getters
> > ##### composition : Composition
> > ##### input : string
> > ##### caret_pos : number
> > ##### commit_notifier : Notifier
> > ##### select_notifier : Notifier
> > ##### update_notifier : Notifier
> > ##### delete_notifier : Notifier
> > ##### option_update_notifier : Notifier
> > ##### property_update_notifier : Notifier
> > ##### unhandled_key_notifier : Notifier
> #### Setters
> > ##### composition : Composition
> > ##### input : string
> > ##### caret_pos : number
### Preedit
> #### Getters
> > ##### text : string
> > ##### caret_pos : number
> > ##### sel_start : number
> > ##### sel_end : number
> #### Setters
> > ##### text : string
> > ##### caret_pos : number
> > ##### sel_start : number
> > ##### sel_end : number
### Composition
> Extends `Segmentation`
> #### Methods
> > ##### empty() : boolean
> > ##### back() : Composition
> > ##### pop_back() : bool
> > ##### push_back(Segmentation)
> > ##### has_finished_composition() : bool
> > ##### get_prompt() : string
### Schema
> #### Functions
> > ##### Schema(schema_id : string) : Schema
> #### Getters
> > ##### schema_id : string
> > ##### schema_name : string
> > ##### config : Config
> > ##### page_size : number 
> > ##### select_keys : string
> #### Setters
> > ##### config : Config
> > ##### select_keys : string
### Config
> #### Methods
> > ##### load_from_file(file_name : string) : boolean
> > ##### save_to_file(file_name : string) : boolean
> > ##### is_null(path : string) : boolean
> > ##### is_value(path : string) : boolean
> > ##### is_list(path : string) : boolean
> > ##### is_map(path : string) : boolean
> > ##### get_bool(path : string) : boolean
> > ##### get_int(path : string) : number
> > ##### get_double(path : string) : number
> > ##### get_string(path : string) : string
> > ##### get_list_size(path : string) : number
> > ##### set_bool(path : string, value : bool) : bool 
> > ##### set_int(path : string, value : number) : boolean
> > ##### set_double(path : string, value : number) : boolean
> > ##### set_string(path : string, value : string) : boolean
### Connection
> #### Methods
> > ##### disconnect()
### Notifier
> #### Methods
> > ##### connect(Notifier, Context) : Connection
### OptionUpdateNotifier
> #### Methods
> > ##### connect(OptionUpdateNotifier, string): Connection
### PropertyUpdateNotifier
> #### Methods
> > ##### connect(PropertyUpdateNotifier, string): Connection
### KeyEventNotifer
> #### Methods
> > ##### connect(KeyEventNotifer, KeyEvent) : Connection
### Log
> Use `Log` to logging your lua function.
> #### Functions
> > ##### info(string)
> > ##### warning(string)
> > ##### error(string)