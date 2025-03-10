# hash-object command

The [hash-object command](https://git-scm.com/docs/git-hash-object) is used to compute the ID value "aka. name" of an object, and sometimes we use it to write the object, although it is not a frequently used command and it is considered as a low-level command still it is for sure an important one and in our implementation version, we'll use this function to do hashing functionality in upcoming commands.  

### What are objects?

#### Object:
- The unit of storage in Git. It is uniquely identified by the SHA-1 hash of its contents. Consequently, an object cannot be changed.  

- Every object has an:
   - **Object name (id)**
     - The unique identifier of an object. The object name is usually represented by a 40-character hexadecimal string. Also colloquially called SHA-1. 
     - That means object names are not just regular file names, it is a unique name calculated depending on the content of the file. This makes sure that all the files are unique from each other. 
     - We conclude that we do not modify files in Git even if you change just one character in a file the name of the file changes too and is stored in a different location "Don't get overridden".
   - **Object type**
     - One of the identifiers "commit", "tree", "tag" or "blob" describes the type of an object.

### Where objects are being stored?

The path where git stores its objects is created by calculating the [SHA-1 hash](https://en.wikipedia.org/wiki/SHA-1) of its contents. Git splits the hash into two parts: the first two characters of the hash, and the rest. Then it uses the first part as the directory name, and the rest as the file name.

```text title="example"
./.mygit/objects/d4/5a25929cd70b0264caea33b209c9cfa845f6bc
```

### How objects are stored?

First, we need to understand the storage format. An object has a header that specifies its type ("commit", "tree", "tag" or "blob"), the size of object contents in bytes, null byte, then the object compressed content, Git uses Zlib for compression.

```text title="example"
blob <size>\0<content>
```

For example, if the file content is `this is file` the blob object file would look like this after decompression.


```text title="example"
blob 12\0this is file
```

#### Object types:
1. Blob: 

    - Stores the content of a file.
    - Contains no metadata like filenames or permissions.
    - Identified by a unique SHA-1 hash.
2. Tree:

    - Represents a directory and its structure.
    - Points to blobs (files) and other trees (subdirectories).
    - Stores filenames and permissions.
3. Commit:

    - Represents a snapshot of the repository.
    - Points to a tree object and parent commits.
    - Includes metadata (author, timestamp, message).
4. Tag:

    - Marks a specific commit, often for releases.
    - Can be lightweight (simple pointer) or annotated (with metadata).

For now, that's all the knowledge we need to start implementing our function.

### Command Implementation

First, we define function `command_hash_object($args)`, the `$args` is an array of arguments that get passed to our function from the `user_call_func` that we used [earlier](.././starting%20code.md) to read user input.

1. Parse arguments:
```php title="./index.php"
$filePath = $args[0];
$writeEnable = $args[1] ?? false;
```

- Extract file path from arguments.
- Check if the second argument is provided to decide if the object should be written to the repo or not.

2. Check if the file exists:
```php title="./index.php"
if (!file_exists($filePath)) {
  echo "Error: File not found.\n";
  return;
}
```

- Check if the file path provided exists and if not we display an error message and return.

3. Read the file:
```php title="./index.php"
$fileContents = file_get_contents($filePath);
```

- Read the file contents and store them in `$fileContents`.

4. Create the object header:
```php title="./index.php"
$header = "blob " . strlen($fileContents) . "\0";
```

- Create the object header by concatenating the following:
  - `blob`: which is the object type.
  - `strlen($fileContents)`: which is the length of the contents inside the file.
  - `\0`: which is the null byte (used to separate the header from the file contents).

5. Concatenate header and contents:
```php title="./index.php"
$fullContent = $header . $fileContents;
```

- concatenate the header with the file contents to get the final storage format  
which is `blob <size>\0<content>`.

6. Compute the SHA-1 hash:
```php title="./index.php"
$hash = sha1($fullContent);
```

- calculate the SHA-1 hash of the object by using [`sha1()`](https://www.php.net/manual/en/function.sha1.php) php built-in function.

7. Write object to repo if writeEnable is true:
```php title="./index.php"
if ($writeEnable) {
  $compressed = gzcompress($fullContent);

  $dir = '.mygit/objects/' . substr($hash, 0, 2);
  if (!is_dir($dir)) {
    mkdir($dir, 0777, true);
  }

  $filePath = $dir . '/' . substr($hash, 2);
  file_put_contents($filePath, $compressed);
}
```

- If the `$writeEnable` value was true:
  - We compress the file content using [`gzcompress()`](https://www.php.net/manual/en/function.gzcompress.php) built-in function which is the compress function for Zlib.
  - We create our directory path `$dir` by concatenating `'.mygit/objects/'` with the first two characters in `$hash`.
  - If the path `$dir` didn't exist we make a new directory using [`mkdir()`](https://www.php.net/manual/en/function.mkdir.php) built-in function.
  - We create our file path `$filePath` by concatenating `$dir` path with `/` and the rest of the file hash. Then we save the compressed content into our file.

8. Output and return the hash:
```php title="./index.php"
echo $hash . PHP_EOL;
return $hash;
```

 - At the end, we output the calculated hash and return it (because we will use this function later in upcoming commands).

```text title="run example"
php ./index.php hash_object test.txt true #if we want it to write
```

```text title="output example"
d0ab265094c565b783e6880d73ff0dbf315d8812
```

Full code for this section:
```php title="index.php"
function command_hash_object($args)
{
  $filePath = $args[0];
  $writeEnable = $args[1] ?? false;

  if (!file_exists($filePath)) {
    echo "Error: File not found.\n";
    return;
  }

  $fileContents = file_get_contents($filePath);

  $header = "blob " . strlen($fileContents) . "\0";

  $fullContent = $header . $fileContents;

  $hash = sha1($fullContent);

  if ($writeEnable) {
    $compressed = gzcompress($fullContent);

    $dir = '.mygit/objects/' . substr($hash, 0, 2);
    if (!is_dir($dir)) {
      mkdir($dir, 0777, true);
    }

    $filePath = $dir . '/' . substr($hash, 2);
    file_put_contents($filePath, $compressed);
  }
  echo $hash . PHP_EOL;
  return $hash;
}
```