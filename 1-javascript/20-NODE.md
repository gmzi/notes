# Node.js

- file system:
  - [reading_files](###reading_files)
  - [writing_files](###writing_files)
- process:
  - [process.env](##process.env)
  - [process.exit](##process.exit)
  - [process.argv](##process.argv)
- modules system:
  - [import/export_files](##import/export_files)
- [import_libraries](##import_libraries)
- [node_callbacks](##node_callbacks)
- [global](##global)
- [npm](##npm)

## file_system_module

### reading_files

`fs.readFile(path, encoding, callback);`

```javascript
const fs = require('fs');

// ASYNCHRONOUS
fs.readFile('words.txt', 'utf8', (err, data) => {
  if (err) {
    console.log('ERROR:', err);
    process.kill(1);
  }
  console.log('File content:', data); // File content: words words words
});

// SYNCHRONOUS:
try {
  // store the read file contents
  var contents = fs.readFileSync('myFile.txt', 'utf8');
  console.log(`file contents are "${contents}"`);
} catch (error) {
  // errors thrown by fs will be caught here
  console.error(error);
  // kill the process and tell the shell it errored
  process.exit(1);
}
```

- path: path to file (relative to working directory)
- encoding: how to interpret file. For text files, this is almost always “utf8”. for binary files (like an image), omit this argument
- callback: function that takes error and data

The fs module is built-in and provides an interface to your local file system.
It is commonly used to read and write files.

### writing_files

`fs.writeFile(path, data, encoding, callback)`

```javascript
const fs = require('fs');

const phrase = 'THIS WILL GO IN THE FILE!';

// OVERWRITE FILE:
fs.writeFile('./files/output.txt', phrase, 'utf8', (err, data) => {
  if (err) {
    console.error(err);
    process.exit(1);
  }
  console.log('Successfully wrote to file!');
});
console.log('writing file...');

// APPEND TO FILE:
fs.appendFile('./files/output.txt', phrase, 'utf8', (err) => {
  if (err) {
    console.log('error', err);
    process.kill(1);
  }
  console.log('it worked');
});

console.log('appending to file...');

// APPEND WITH FLAG:
fs.writeFile(
  './files/output.txt',
  phrase,
  { encoding: 'utf8', flag: 'a' },
  (err) => {
    if (err) {
      console.log('error', err);
      process.kill(1);
    }
    console.log('it worked');
  }
);
console.log('appending to file...');
```

## process.env

Create .env variables:
in terminal:

- `export SECRET_INFO='abc123'`

Access .env variables in file:

```javascript
console.log(process.env.SECRET_INFO); // abc123
```

Access .env variables in terminal:

1. run node
2. `process.SECRET_INFO`

process.env is an object that contains key-value pairs. The keys are the names of the environment variables, and the values are... well, it's values.

For secret keys and APIs credentials.

Is a global object for managing the current script. Using process, we can:

- Access environment variables.
- See the command-line arguments passed to the script.
- Kill the script.

## process.exit

```javascript
console.log('caquite'); // this will run

process.exit(1);

console.log('lfdslasdfsa'); // this won't run
```

Exit the program inmediately, and displays a code:
Standard Codes:

- success code: 0
- failure code: 1 and above

You can assign your own meaning to codes.

## process.argv

An array of command-line arguments given to be executed by the program. First two are paths to node in local machine and to file, then comes the rest of the arguments:

In terminal:

```
$ node showArgs.js hello world
    0 '/path/to/node' -->
    1 '/path/to/showArgs.js'
    - whatever is in the script.js file to run
    - 'hello'
    - 'world'
```

In js:

```javascript
const fs = require('fs');
const process = require('process');

const myPath = process.argv[2];

fs.readFile(myPath, 'utf8', (err, data) => {
  if (err) {
    console.log('LA RECONCHA TUYA!!!!!', err);
    process.kill(1);
  }
  console.log('Heres the file: ', data);
});
```

[process_docs](https://nodejs.org/api/process.html#process_process)

## import/export_files

Double process: export from a file, import it in the other.

```javascript
// EXPORT from /helpers.js:
function add(x, y) {
  return x + y;
}
function subtract(x, y) {
  return x - y;
}

module.exports = {
  add: add,
  color: 'red',
}; // not exporting 'subtract' function

// --------------------------------------------------------------------

// IMPORT in /app.js:
// import the module:
const { add, color } = require('./helpers'); //mind the './' for same folder, '../' for one folder up.
// call the methods:
color; // 'red'
add(1, 1); // 2
```

Modules are the way to share code across different files in a Node project.
To share code accress different files, have to export/import manually.

## import_libraries

```javascript
// Import libraries:
const axios = require('axios');
const faker = require('faker');

// use libraries:
console.log(faker.name.findName());
console.log(axios);
```

## node_callbacks

Node callbacks use Error-first pattern: first param is error, node will supply an error object if there's an error, otherwise null as arguments.

```javascript
const fs = require('fs');

fs.readFile('myFile.txt', 'utf8', function (err, data) {
  if (err) {
    // handle error, options:
    // - log the error to the console.
    // - exit the program with process.exit(1)
  }
  // otherwise we're good, continue execution
});
```

## global

- _global object_ is in node.js the equivalent to browser's window object.
- no DOM methods nor document methods in node.js
- node.js has file system and can start server processes.
- There are libraries that can run in both node and browsers js (like axios)

## npm

[npm](https://www.npmjs.com)
Node package manager
Massive registry of add-on libraries
Command line tool, npm, comes with Node
Easy dependency management for a project
Register with npm to publish open-source or proprietary packages

Javascript environment that runs server-side (not in the browser). Uses Chrome 8 engine, but doesn't require Chrome.
With Node.js, entire stack can be made in JS, back and front end. Has a lot of npm packages. Very efficient in streamming. Can use Electron to "translate" your js app into any language.
Not so good for very complex server-side computation.

- `npm install nameOfPackage`
  - ex. `npm install axios`