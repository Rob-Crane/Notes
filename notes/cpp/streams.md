# C++ Streams
* IO Streams is a general input/output facility for C++ and generally considered one of its strengths.  Internally, streams are series of characters and represent a serial, character-based interface to any storage facility (storage, buffers, files, etc.).
* Built-in data types work by default and user-defined data types can be made to work by overloading the insertion (>>) and extraction (<<) operator.  These overloaded operations define the relationship between the user-defined type and the raw character stream.
* Can't access random portion of a stream but can seek to a position.
* Under the hood, the streambuf is a character buffer which provides the device-specific interface for streams to use.
* When a stream is initialized, a _put_ and _get_ pointer is intialized that represents the position at write and read operations occur.  The `seekg()`, `seekp()`, `tellg()`, and `tellp()` functions move or return the value of these pointers.

## Error Handling
* The validity of a stream can be checked by using as a boolean:
  ```cpp
  ifstream file_stream("file.txt");
  if(!file_stream) {
    cerr<<"Error!"<<endl;
  }
  ```
* A stream input or ouput operation returns a reference to the stream itself so the success of the operation can be checked by by using the operation itself as a boolean: `if(cin >> x) ...`
* Following functions also describe state of stream:
  * `good()` - everything OK
  * `bad()` - fatal error occured
  * `fail()` - last operation failed, stream still up
  * `eof()` - end of file reached
  
