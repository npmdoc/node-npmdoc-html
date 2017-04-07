# api documentation for  [html (v1.0.0)](https://github.com/maxogden/commonjs-html-prettyprinter)  [![npm package](https://img.shields.io/npm/v/npmdoc-html.svg?style=flat-square)](https://www.npmjs.org/package/npmdoc-html) [![travis-ci.org build-status](https://api.travis-ci.org/npmdoc/node-npmdoc-html.svg)](https://travis-ci.org/npmdoc/node-npmdoc-html)
#### HTML pretty printer CLI utility (based on jsbeautifier)

[![NPM](https://nodei.co/npm/html.png?downloads=true)](https://www.npmjs.com/package/html)

[![apidoc](https://npmdoc.github.io/node-npmdoc-html/build/screenCapture.buildNpmdoc.browser._2Fhome_2Ftravis_2Fbuild_2Fnpmdoc_2Fnode-npmdoc-html_2Ftmp_2Fbuild_2Fapidoc.html.png)](https://npmdoc.github.io/node-npmdoc-html/build/apidoc.html)

![npmPackageListing](https://npmdoc.github.io/node-npmdoc-html/build/screenCapture.npmPackageListing.svg)

![npmPackageDependencyTree](https://npmdoc.github.io/node-npmdoc-html/build/screenCapture.npmPackageDependencyTree.svg)



# package.json

```json

{
    "author": {
        "name": "Max Ogden",
        "email": "max@maxogden.com",
        "url": "http://maxogden.com"
    },
    "bin": {
        "html": "./bin/html.js"
    },
    "bugs": {
        "url": "https://github.com/maxogden/commonjs-html-prettyprinter/issues"
    },
    "contributors": [
        {
            "name": "Nochum Sossonko",
            "email": "nsossonko@hotmail.com"
        },
        {
            "name": "Einar Lielmanis",
            "email": "elfz@laacz.lv"
        }
    ],
    "dependencies": {
        "concat-stream": "^1.4.7"
    },
    "description": "HTML pretty printer CLI utility (based on jsbeautifier)",
    "devDependencies": {},
    "directories": {},
    "dist": {
        "shasum": "a544fa9ea5492bfb3a2cca8210a10be7b5af1f61",
        "tarball": "https://registry.npmjs.org/html/-/html-1.0.0.tgz"
    },
    "gitHead": "0717eb216ebcc67399ca3c0406e5c03a77f7e761",
    "homepage": "https://github.com/maxogden/commonjs-html-prettyprinter",
    "keywords": [
        "html",
        "tabifier",
        "beautifier",
        "prettyprinter",
        "prettifier",
        "pretty",
        "command",
        "shell"
    ],
    "license": "BSD",
    "main": "lib/html.js",
    "maintainers": [
        {
            "name": "maxogden",
            "email": "max@maxogden.com"
        }
    ],
    "name": "html",
    "optionalDependencies": {},
    "readme": "ERROR: No README data found!",
    "repository": {
        "type": "git",
        "url": "git+https://github.com/maxogden/commonjs-html-prettyprinter.git"
    },
    "scripts": {
        "test": "echo \"Error: no test specified\" && exit 1"
    },
    "version": "1.0.0"
}
```



# <a name="apidoc.tableOfContents"></a>[table of contents](#apidoc.tableOfContents)

#### [module html](#apidoc.module.html)
1.  [function <span class="apidocSignatureSpan">html.</span>prettyPrint (html_source, options)](#apidoc.element.html.prettyPrint)



# <a name="apidoc.module.html"></a>[module html](#apidoc.module.html)

#### <a name="apidoc.element.html.prettyPrint"></a>[function <span class="apidocSignatureSpan">html.</span>prettyPrint (html_source, options)](#apidoc.element.html.prettyPrint)
- description and source-code
```javascript
function style_html(html_source, options) {
//Wrapper function to invoke all the necessary constructors and deal with the output.

  var multi_parser,
      indent_size,
      indent_character,
      max_char,
      brace_style,
      unformatted;

  options = options || {};
  indent_size = options.indent_size || 4;
  indent_character = options.indent_char || ' ';
  brace_style = options.brace_style || 'collapse';
  max_char = options.max_char == 0 ? Infinity : options.max_char || 70;
  unformatted = options.unformatted || ['a', 'span', 'bdo', 'em', 'strong', 'dfn', 'code', 'samp', 'kbd', 'var', 'cite', 'abbr', '
acronym', 'q', 'sub', 'sup', 'tt', 'i', 'b', 'big', 'small', 'u', 's', 'strike', 'font', 'ins', 'del', 'pre', 'address', 'dt', '
h1', 'h2', 'h3', 'h4', 'h5', 'h6'];

  function Parser() {

    this.pos = 0; //Parser position
    this.token = '';
    this.current_mode = 'CONTENT'; //reflects the current Parser mode: TAG/CONTENT
    this.tags = { //An object to hold tags, their position, and their parent-tags, initiated with default values
      parent: 'parent1',
      parentcount: 1,
      parent1: ''
    };
    this.tag_type = '';
    this.token_text = this.last_token = this.last_text = this.token_type = '';

    this.Utils = { //Uilities made available to the various functions
      whitespace: "\n\r\t ".split(''),
      single_token: 'br,input,link,meta,!doctype,basefont,base,area,hr,wbr,param,img,isindex,?xml,embed,?php,?,?='.split(','), //
all the single tags for HTML
      extra_liners: 'head,body,/html'.split(','), //for tags that need a line of whitespace before them
      in_array: function (what, arr) {
        for (var i=0; i<arr.length; i++) {
          if (what === arr[i]) {
            return true;
          }
        }
        return false;
      }
    }

    this.get_content = function () { //function to capture regular content between tags

      var input_char = '',
          content = [],
          space = false; //if a space is needed

      while (this.input.charAt(this.pos) !== '<') {
        if (this.pos >= this.input.length) {
          return content.length?content.join(''):['', 'TK_EOF'];
        }

        input_char = this.input.charAt(this.pos);
        this.pos++;
        this.line_char_count++;

        if (this.Utils.in_array(input_char, this.Utils.whitespace)) {
          if (content.length) {
            space = true;
          }
          this.line_char_count--;
          continue; //don't want to insert unnecessary space
        }
        else if (space) {
          if (this.line_char_count >= this.max_char) { //insert a line when the max_char is reached
            content.push('\n');
            for (var i=0; i<this.indent_level; i++) {
              content.push(this.indent_string);
            }
            this.line_char_count = 0;
          }
          else{
            content.push(' ');
            this.line_char_count++;
          }
          space = false;
        }
        content.push(input_char); //letter at-a-time (or string) inserted to an array
      }
      return content.length?content.join(''):'';
    }

    this.get_contents_to = function (name) { //get the full content of a script or style to pass to js_beautify
      if (this.pos == this.input.length) {
        return ['', 'TK_EOF'];
      }
      var input_char = '';
      var content = '';
      var reg_match = new RegExp('\<\/' + name + '\\s*\>', 'igm');
      reg_match.lastIndex = this.pos;
      var reg_array = reg_match.exec(this.input);
      var end_script = reg_array?reg_array.index:this.input.length; //absolute end of script
      if(this.pos < end_script) { //get everything in between the script tags
        content = this.input.substring(this.pos, end_script);
        this.pos = end_script;
      }
      return content;
    }

    this.record_tag = function (tag){ //function to record a tag and its parent in this.tags Object
      if (this.tags[tag + 'count']) { //check for the existence of this tag type
        this.tags[tag + 'count']++;
        this.tags[tag + this.tags[tag + 'count']] = th ...
```
- example usage
```shell
n/a
```



# misc
- this document was created with [utility2](https://github.com/kaizhu256/node-utility2)
