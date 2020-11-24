# Introduction

**Welcome to librime-lua**

This page would teach you how to compose a lua script for librime

## Export Lua function to librime 

### Create rime.lua
---
Please Create Rime.lua at `PATH_TO_RIME_USER_DATA_DIR/rime.lua`
This is the only entry for librime to access lua component, All function in `rime.lua` would exposed to librime.

---

### Import function to rime.lua

***rime.lua*** *All lua function Example are in `librime-lua-src/example`*

```lua
-- translators:
	-- date_translator Details at `lua/date.lua`
	-- Translate `date` to current date.
	date_translator = require("date")

	-- time_translator Details at `lua/time.lua`
	-- Translate `time` to current time.
	time_translator = require("time")

	-- number_translator Details at `lua/number.lua`
	-- translate '/' + Digits to Digits in Han Character
	number_translator = require("number")
-- Filters:

	-- charset_filter Details at `lua/charset.lua`
	-- charset_filter； filter for CJK Extended Candidate
	-- charset_comment_filter: Add Comment for Specified charset
	local charset = require("charset")
	charset_filter = charset.filter
	charset_comment_filter = charset.comment_filter

	-- Details at `lua/single_char.lua`
	-- single_char_filter: Bringing Candidte with Single Character to first place by sorting Candidate
	single_char_filter = require("single_char")

	-- reverse_lookup Details at `lua/reverse.lua`
	-- ADD PINYIN comment to terra_pinyin
	reverse_lookup_filter = require("reverse")
```

### Bind lua Function to rime schema

Write something below on Rime schema

```yaml
engine:
  translators:
    - lua_translator@date
    - lua_translator@time
  filters:
    - lua_filter@charset
    - lua_filter@single_char_filter

```

Then Rime Engine would able to call lua Function.

# Introducing built-in librime-lua Component!

> Check out **API** page for more details!