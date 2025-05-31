# SExtractor
Extract and import text from GalGame scripts (most require plaintext)

## Python Dependencies:
Python version 3.9 or higher required (Python 3.11 recommended)
* pyqt5
* colorama
* pandas
* python-rapidjson

## Detailed Usage Tutorials
* [julixian](https://www.ai2.moe/topic/28969-sextractor%E4%BD%BF%E7%94%A8%E5%BF%83%E5%BE%97)
* [无聊荒芜](https://www.ai2.moe/topic/29048-sextractor%E4%BD%BF%E7%94%A8%E2%80%94%E2%80%94%E8%BF%9B%E9%98%B6%E8%AF%B4%E6%98%8E%E5%8A%A0%E9%83%A8%E5%88%86%E5%BC%95%E6%93%8E%E7%A4%BA%E4%BE%8B%EF%BC%88%E7%9C%8B%E5%AE%8Csextractor%E4%BD%BF%E7%94%A8%E5%BF%83%E5%BE%97%E5%86%8D%E6%9D%A5%EF%BC%89)

## Supported Engines:
Different games of the same engine may have different formats. Please refer to the examples within the program.
* TXT plaintext (regex matching. Optional utf-8, utf-8-sig, utf-16(LE BOM))
* BIN binary text (regex matching. Default reads shift-jis, writes GBK)
* JSON text (regex matching, only searches values, if value is empty, automatically copies key to value)
* ANIM
* AZ System (Encrypt Isaac)
* Artemis
* Black Rainbow
* CSV
* CScript (has precompiled modules, Python 3.11 recommended)
* Cyberworks / CSystem
* EAGLS
* FVP
* Kaguya
* Krkr
* MED (DxLib modified)
* MoonHir
* NekoSDK
* RPG Maker MV
* RPG Maker VX Ace
* RealLive
* RenPy
* SystemC
* WillPlus
* Yu-ris

## Other Features
* Can export VNT's JIS tunnel file `sjis_ext.bin`, requires use with [VNTProxy](#related-projects). (Also exports UIF configuration)
* Can export UIF's JIS replacement configuration `uif_config.json`, requires use with [UniversalInjectorFramework](#related-projects).
* JIS replacement fonts are available under `Tools/Font` for use when dll cannot hook the game.
* All custom `config*.ini` files in the folder will be read, where * cannot start with a number. (Example: `configTest.ini`)
* `text_conf.json` configures text processing. Priority reads configuration from ctrl files in working directory; if none exists, reads default configuration from tool root directory.
```
text_conf.json:
  "replace_before_split" - Replace before splitting
  "trans_replace" - Translation replacement, limited by import encoding
  "orig_replace" - Original text replacement and restoration
  "name_replace" - Original text replacement and restoration for names only
```

## Current Regex Presets
<font color=red>(More preset regex patterns can be found in root directory `预设正则.fake.ini`)</font>
* AST
* Artemis
* Cyberworks_JIS
* CSV_Livemaker
* EntisGLS
* Krkr
* Nexas
* RealLive (extract options separately)
* RPGMV_System
* RPGVX_NotMap
* SFA_AOS
* Valkyria_dat_txt
* Valkyria_ODN
* Yuris_txt (non-ybn)
* BIN brute force matching
* Two-line TXT
* Export all (mostly for format conversion)
* Custom rules (auto-save)
* None (restore to engine default)

## Tools
* AST: arc2 packaging
* Astronauts: gpx packaging, Mwb extraction
* AZ System: isaac encryption
* BlackRainbow: packaging
* CScript: packaging, compression/decompression
* Cyberworks: UTF-16 unpackaging
* DxLib: unpacking
* EAGLS: unpackaging
* Font: Fonts generated from JIS replacement dictionary
* Malie: packaging
* RealLive: unpackaging, secondary encryption/decryption
* SHook: shell jumping, Loader
* Silky: Azurite packaging
* Unity: data.dsm encryption/decryption
* UniversalInjectorFramework: dll

## Regex-Related Instructions
File reading methods are divided into two main categories: `txt` and `bin`. The former processes as strings, the latter as bytes.
* `separate=reg` - Separator in bin mode, default is `separate=\r\n`. During import, the separate string `\r\n` will be added; if it contains capture groups like `separate=([\x01-\x02]\x00|\x00)`, the separator will be extracted.
* `startline=0` - Starting line number for each file processing; default is 0.
* `structure=paragraph` - Extraction structure. When set to `paragraph`, it processes group names other than name or msg, such as `unfinish`. (Not all engine regex support this; `TXT` and `BIN` engines definitely support it)
* `extraData=data` - Custom parameters for engines, refer to each engine's default regex for specific usage.
* `ignoreDecodeError=1` - In bin mode, ignore decode encoding errors during text extraction.
* `checkJIS=reg` - In bin mode, check if bytes conform to shift-jis encoding. Default allows only double-byte characters; `reg` specifies supported single-byte characters. For example, `checkJIS=[\n]` allows newline characters.
* `postSkip=reg` - During extraction, perform `re.search(reg, text)` matching on extracted text. If regex matches successfully, ignore that text and don't export it. For example, `postSkip=^[0-9]` ignores text starting with numbers.
* `sepStr=reg` - Used only by Krkr_Reg engine, indicates separator matching; default is `sepStr=[^\[\]]+`, meaning split by square brackets.
* `endStr=reg` - Used only by Krkr_Reg engine, indicates paragraph end matching.
* `ctrlStr=reg` - Used only by Krkr_Reg engine, indicates control segments to skip. (Similar to general postSkip)
* `version=0` - Mainly used by Yuris, indicates file structure version
* `decrypt=auto` - Mainly used by Yuris, indicates decryption. Auto means automatic detection; can be forced, e.g., `decrypt=\xD3\x6F\xAC\x96`. Delete this line if already decrypted.
* `pureText=1` - Equivalent to checking `Enable pure text regex mode for BIN`
* `writeOffset=1` - Mainly used by CSV, offset write column to the right.

### Regex Examples
Each line of text is matched from top to bottom. (Both skip and search successful matches will interrupt and not perform subsequent regex matching)
```
00_skip=^error
10_search=^(?P<name>Name.*)$
20_search=^(?P<pre_name>「.+」)$
21_search=^(?P<pre_nameANDunfinish>「.*)$
25_search=^(.+?)(?<=」|。)$
26_search=^(?P<unfinish>.+?)$
postSkip=^[0-9]
structure=paragraph
```
* 00 Skip lines starting with `error`. Skip interrupts paragraph structure (using postSkip to handle error wouldn't interrupt)
* 10 Extract lines starting with `Name` and specify itself as `name` (`name` defaults to `predel_unfinish`)
* 20 Extract lines with `「」` and specify previous line as `name`
* 21 Extract lines starting with `「` and specify previous line as `name`, itself as `unfinish`
* 25 Extract lines ending with `」`
* 26 Extract lines with any characters (. doesn't include newlines)
* postSkip Skip if starting with numbers, doesn't interrupt paragraph structure
* Finally merge text in order. If it's `unfinish`, add \r\n and don't switch to next message.
* Group names `pre_` and `predel_` can be freely combined with `name` and `unfinish`. `AND` can also have any number.
* Original txt:
```
Text0
Name1
Text1。
MaybeName2
「Text2」
MaybeName3
「
Text3
33text
Text333
error
」
```
* Extracted as:
```
[
  {
    "message": "Text0"
  },
  {
    "name": "Name1",
    "message": "Text1。"
  },
  {
    "name": "MaybeName2",
    "message": "「Text2」"
  },
  {
    "name": "MaybeName3",
    "message": "「\r\nText3\r\nText333"
  },
  {
    "message": "」"
  }
]
```

## Supported Export Formats:
* json dictionary { text : "" }
* json dictionary { text : text }
* json list [ { name : name, message : dialogue with line breaks } ]
* json dictionary { text with line breaks : "" }
* json dictionary { text with line breaks : text with line breaks }
* txt document { text }
* txt document [ text with line breaks ]
* json list [ text with line breaks ]

## Related Projects
1. [game_translation](https://github.com/ssynn/game_translation)
2. [SiglusTools](https://github.com/yanhua0518/GALgameScriptTools)
3. [CSystemTools](https://github.com/arcusmaximus/CSystemTools)
4. [VNTranslationTools](https://github.com/arcusmaximus/VNTranslationTools)
5. [UniversalInjectorFramework](https://github.com/AtomCrafty/UniversalInjectorFramework)
6. [GalTransl_DumpInjector](https://github.com/XD2333/GalTransl_DumpInjector)
7. [EAGLS](https://github.com/jszhtian/EAGLS)
8. [MalieTools](https://github.com/Dir-A/MalieTools)
9. [Garbro fork](https://github.com/satan53x/GARbro)
