# cat-file command

The [cat-file command](https://git-scm.com/docs/git-cat-file) is used to read and display the contents or other data of a Git object, given its hash. This helps inspect stored objects within the repository.

In Git there is a lot of options using the cat-file command, and usually we pass a short version of the hash to it, but for simplicity, in this project we'll pass the full hash and print all the file data.

We already learned about objects in the last section so we can start implementing the function now.

### Command Implementation

First, we define the function `command_cat_file($args)`.

1. Get object hash:

```php title="./index.php"
$hash = (string) $args[0];
```

  - We get the hash from `$args[0]` and save it as a string so we can operate on it.

2. Get the directory and file paths:

```php title="./index.php"
$dir = '.mygit/objects/' . substr($hash, 0, 2);
$file = $dir . '/' . substr($hash, 2);
```

  - As we did in the hash-object command, we create our directory path `$dir` by concatenating `'.mygit/objects/'` with the first two characters in `$hash`.
  - We create our file path `$file` path by concatenating `$dir` path with `/` and the rest of the file hash.

3. Check if the file exists:
```php title="./index.php"
if (!file_exists($file)) {
  echo "Error: Object not found.\n";
  return;
}
```

  - Check if the file path provided exists and if not we display an error message and return.

4. Get compressed data:

```php title="./index.php"
$compressed = file_get_contents($file);
```

  - We get the data of the file which is currently compressed.

5. Uncompress data:

```php title="./index.php"
$data = gzuncompress($compressed);
```

  - We uncompress data using [`gzuncompress()`](https://www.php.net/manual/en/function.gzuncompress.php) built-in function which is the uncompress function for Zlib.

6. Validate data:

```php title="./index.php"
if ($data === false) {
  echo "Error: Failed to decompress the object.\n";
  return;
}
```

  - If `$data` is false that means the `gzuncompress()` function failed to uncompress the data and return false so we output an error message to the user and return.

7. Get file contents:

```php title="./index.php"
$parts = explode("\0", $data, 2);
if (count($parts) < 2) {
  echo "Error: Invalid MyGit object format.\n";
  return;
}

$fileContent = $parts[1];

echo $fileContent;
```

  - Remember the null byte `\0` we used earlier in the hash-object command here is where it comes in handy, we use it to separate the object data (`<type> <size>`) from the actual content that we want.
  - We use the `explode()` built-in function to separate `$data` string using the separator `\0` into an array.
  - Check if the number of elements in array `$parts` is less than 2 which means there was something wrong with this object and we output an error message to the user and return.
  - Take the actual data that we want which is at index 1 in `$parts` and store it in `$fileContent` then we output it.

```text title="run example"
php ./index.php cat_file d0ab265094c565b783e6880d73ff0dbf315d8812
```

```text title="output example"
this is test text
```

Full code for this section:
```php title="./index.php"
function command_cat_file($args)
{
  $hash = (string) $args[0];

  $dir = '.mygit/objects/' . substr($hash, 0, 2);
  $file = $dir . '/' . substr($hash, 2);

  if (!file_exists($file)) {
    echo "Error: Object not found.\n";
    return;
  }

  $compressed = file_get_contents($file);

  $data = gzuncompress($compressed);

  if ($data === false) {
    echo "Error: Failed to decompress the object.\n";
    return;
  }

  $parts = explode("\0", $data, 2);
  if (count($parts) < 2) {
    echo "Error: Invalid MyGit object format.\n";
    return;
  }

  $fileContent = $parts[1];

  echo $fileContent;
}
```