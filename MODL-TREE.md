<h1>🌳 <strong>MODL‑TREE — Official Specification &amp; Guide</strong></h1><p></p><p><strong>Modular Oriented Data Language — Tree Recursive Expression Encoding</strong> <br>Version 1.0 (Foundational Specification)</p><p></p><h1>1. Overview</h1><p></p><p>MODL‑TREE is a <strong>module‑oriented, depth‑encoded, symbol‑driven hierarchical data language</strong> designed for:</p><p></p><ul><li><p>human readability</p><p></p></li><li><p>machine parsability</p><p></p></li><li><p>recursive tree structures</p><p></p></li><li><p>modular organization</p><p></p></li><li><p>symbolic clarity</p><p></p></li></ul><p></p><p>Its core idea:<br><strong>Depth is encoded by dash count, and structure is expressed through directional operators and symmetric blocks.</strong></p><p></p><p>MODL‑TREE files always have:</p><p></p><ul><li><p><strong>one root module</strong></p><p></p></li><li><p><strong>nested submodules</strong></p><p></p></li><li><p><strong>optional blocks</strong></p><p></p></li><li><p><strong>constants</strong></p><p></p></li><li><p><strong>key/value entries</strong></p><p></p></li></ul><p></p><h1>2. File Structure</h1><p></p><h2>2.1 File Header &amp; Footer</h2><p></p><p>A MODL‑TREE file begins and ends with a <strong>boundary marker</strong>:</p><p></p><p>Code</p><pre><code class="language-auto">( Name Of Root )+-----+-----+-----+-----+-----+
...
+-----+-----+-----+-----+-----+( Name Of Root )
</code></pre><p>These markers:</p><p></p><ul><li><p>visually frame the file</p><p></p></li><li><p>declare the root module name</p><p></p></li><li><p>ensure the file is complete</p><p></p></li></ul><p></p><p>Each boundary is a <strong>file boundary marker</strong>.</p><p></p><h1>3. Depth Encoding</h1><p></p><h2>3.1 Dash‑Count Rule</h2><p></p><p>Depth is determined by the number of leading dashes:</p><p></p><ul><li><p><code>-</code> → depth 1</p><p></p></li><li><p><code>--</code> → depth 2</p><p></p></li><li><p><code>---</code> → depth 3</p><p></p></li><li><p>and so on</p><p></p></li></ul><p></p><p>This is the <strong>depth encoding system</strong>.</p><p></p><p>Indentation is optional; the parser relies <strong>only</strong> on dash count.</p><p></p><h1>4. Directional Operators</h1><p></p><h2>4.1 <code>/</code> — <em>Nesting operator</em></h2><p></p><p>Meaning: <strong>This module contains deeper modules.</strong></p><p></p><p>Example:<br><code>--/( CLI/TUI )</code></p><p></p><p>This is a <strong>nesting operator</strong>.</p><p></p><h2>4.2 <code>\</code> — <em>Single‑item operator</em></h2><p></p><p>Meaning: <strong>This entry does not nest further.</strong></p><p></p><p>Example:<br><code>---\const 'server start' = "Runs the server"</code></p><p></p><p>This is a <strong>single‑item operator</strong>.</p><p></p><h1>5. Module Syntax</h1><p></p><h2>5.1 Nest Holder <code>( … )</code></h2><p></p><p>Modules are named using parentheses:</p><p></p><p>Code</p><pre><code>-/( Docs )
</code></pre><p>This is a <strong>module holder</strong>.</p><p></p><h2>5.2 Module Forms</h2><p></p><p>There are three module forms:</p><p></p><h3>A. Nesting Module</h3><p></p><p>Code</p><pre><code>--/( ModuleName )
</code></pre><h3>B. Single‑Entry Module</h3><p></p><p>Code</p><pre><code>--\ 'Key' = Value
</code></pre><h3>C. Block Module</h3><p></p><p>Code</p><pre><code>--/ Name { ... }--Name
</code></pre><p>This is a <strong>[block module</strong>.</p><p></p><h1>6. Block Structures</h1><p></p><h2>6.1 Multi‑Object Block</h2><p></p><p>Starts with:</p><p></p><p>Code</p><pre><code>--/ Name {
</code></pre><p>Ends with:</p><p></p><p>Code</p><pre><code>}--Name
</code></pre><p>This block <strong>does not nest further</strong> but contains multiple entries.</p><p></p><p>This is a <strong>[flat block</strong>.</p><p></p><h2>6.2 Constant‑Only Block</h2><p></p><p>Starts with:</p><p></p><p>Code</p><pre><code>--/ Name const{
</code></pre><p>Contains only <strong>keys</strong>, no values.</p><p></p><p>This is a <strong>[const‑only block</strong>.</p><p></p><h1>7. Constants</h1><p></p><h2>7.1 Standard Constant</h2><p></p><p>Code</p><pre><code>const Key = Value
</code></pre><h2>7.2 Multi‑Constant (non‑block)</h2><p></p><p>Code</p><pre><code>--\const 'Command' = "Meaning"
</code></pre><p>This is a <strong>multi‑constant entry</strong>.</p><p></p><h1>8. Key/Value Entries</h1><p></p><h2>8.1 Inside Blocks</h2><p></p><p>Entries begin with <code>**</code>.</p><p></p><h3>Key/value form:</h3><p></p><p>Code</p><pre><code>** CPU = AMD  \
</code></pre><h3>Key‑only form:</h3><p></p><p>Code</p><pre><code>** Main  \
</code></pre><p>The trailing <code>\</code> marks the end of the entry.</p><p></p><p>This is a <strong>block entry</strong>.</p><p></p><h1>9. Comments</h1><p></p><h2>9.1 Comment Syntax</h2><p></p><p>Comments use:</p><p></p><p>Code</p><pre><code>$*{ ... }*$
</code></pre><p>They may appear anywhere outside of structural symbols.</p><p></p><p>This is a <strong>comment block</strong>.</p><p></p><h1>10. Grammar (EBNF‑style)</h1><p></p><p>Code</p><pre><code>file            = header , module* , footer ;

header          = "( " , name , " )+-----+-----+-----+-----+-----+" ;
footer          = "+-----+-----+-----+-----+-----+( " , name , " )" ;

module          = depth , ( nesting-module | single-entry | block-module ) ;

depth           = "-" , { "-" } ;

nesting-module  = "/( " , name , " )" ;

single-entry    = "\" , ( const-entry | kv-entry ) ;

block-module    = "/ " , name , block-type , "{" ,
                   block-entry* ,
                   "}" , "--" , name ;

block-type      = "" | " const" ;

block-entry     = "** " , ( key-only | key-value ) , "\" ;

const-entry     = "const " , key , "=" , value ;

kv-entry        = key , "=" , value ;

key-only        = key ;
key-value       = key , "=" , value ;

name            = string ;
key             = string ;
value           = string ;
string          = quoted-string | unquoted-string ;
</code></pre><p>This is the <strong>[formal grammar</strong>.</p><p></p><h1>11. Full Example (Annotated)</h1><p></p><p>Code</p><pre><code>( Web Server - Servere )+-----+-----+-----+-----+-----+
   -/( Docs )
      --/( For )
         ---/ Devices {
            ** CPU = AMD  \
            ** Bits = 64  \
            ** User = "Someone who wants to serve stuff to the web!"  \
         }---Devices
         ---/[ People {
            ** Developers = True  \
            ** "Anyone who wants to start a web server" = True  \
            ** "Taxi Driver" = False  \
         }---People
      --/( CLI/TUI )
         ---\const 'servere start file.servere' = "Reads the config file and starts the server"
         ---\const 'server kill file.servere' = "Reads the config file and kills the server for it"
      --\ 'Other Info' = "Read out docs at :URL:https://docs.webservers.com/servere:URL: for other info"
   -/( Executable Binary Files )
      --/[ Parser const{
         ** Main  \
         ** Parser  \
         ** Validator  \
         ** Tokenizer  \
      }--Parser
      --/[ CLI/TUI const{
         ** Start  \
         ** Kill  \
      }--CLI/TUI
      --/[ Servers const{
         ** HTTP  \
         ** OpenSSL  \
         ** CustomHandler  \
         ** OtherHandler  \
      }--Servers
+-----+-----+-----+-----+-----+( Web Server - Servere )
</code></pre><p>Every part of this example is a <strong>[MODL‑TREE structural element</strong>.</p><p></p><h1>12. Design Philosophy</h1><p></p><p>MODL‑TREE is built on:</p><p></p><ul><li><p><strong>modularity</strong></p><p></p></li><li><p><strong>symbolic clarity</strong></p><p></p></li><li><p><strong>recursive structure</strong></p><p></p></li><li><p><strong>human readability</strong></p><p></p></li><li><p><strong>machine determinism</strong></p><p></p></li></ul><p></p><p>It is ideal for:</p><p></p><ul><li><p>configuration files</p><p></p></li><li><p>documentation trees</p><p></p></li><li><p>modular software systems</p><p></p></li><li><p>hierarchical metadata</p><p></p></li><li><p>interpreters and compilers</p><p></p></li><li><p>Databases</p></li></ul><p></p><ul><li><p>Anything Else</p></li></ul>
