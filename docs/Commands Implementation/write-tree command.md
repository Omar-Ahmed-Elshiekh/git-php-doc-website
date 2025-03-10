# write-tree command

The [write-tree command](https://git-scm.com/docs/git-write-tree) is used to create a tree object from the current index file (staging area). As we said earlier, a tree object represents the directory structure of a repository at a specific point in time.  
Git processes the current index and writes a tree object that can be referenced later by a commit. The output is a SHA-1 hash that uniquely identifies the tree object.

### Command Implementation

1. Check if index file exists:

```php title="./index.php"
if (!file_exists('.mygit/index')) {
  echo "Error: No index file found\n";
  return;
}
```

- Check if the index file doesn't exist and output an error message.

2. Reading and validating the index file:

This part ensures that only valid file entries are processed before generating the tree object.

```php title="./index.php"
$entries = [];
$index_content = file_get_contents('.mygit/index');

if($index_content === ''){
  echo "Error: Index is empty\n";
  return;
}

$lines = explode("\n", trim($index_content));
foreach ($lines as $line) {
  if (empty(trim($line)))
    continue;

  $parts = explode(" ", trim($line));
  if (count($parts) < 4)
    continue;

  $entries[] = $parts;
}

$tree_content = '';
foreach ($entries as $entry) {
  $tree_content .= implode(" ", $entry) . "\n";
}
```

- We start by reading the index file contents and storing them in `$index_contents`.
- Check if the index file is empty, output an error, and return if it is.
- We split the index contents into lines.
- We iterate over each line and validate that it is not empty and we skip this iteration if it is.
- Then we split each line into parts to validate that each line has the correct data format which is `<mode> <type> <hash> <path>`.
- Then we append the `$path` array to `$entries`.
- At the end, we reconstruct the tree content again by concatenating each entry to `$tree_content`.

3. Add a tree object header:

```php title="./index.php"
$header = "tree " . strlen($tree_content) . "\0";
$full_content = $header . $tree_content;
```

- We make our header by concatenating tree with length of the tree content and null byte (we explained Git storage format earlier).
- Then we concatenate the header with the tree content to get the full content that will be stored.

4. Compute the sha-1 hash:

```php title="./index.php"
$hash = sha1($full_content);
```

- A SHA-1 hash is computed for the full tree object which uniquely identifies the object.

5. Compress the tree object:

```php title="./index.php"
$compressed = gzcompress($full_content);
```

- Compress the tree content with Zlib compression.

6. Store the tree object:

```php title=./index.php"
$dir = ".mygit/objects/" . substr($hash, 0, 2);
if (!is_dir($dir)) {
  mkdir($dir, 0777, true);
}

$path = $dir . "/" . substr($hash, 2);
file_put_contents($path, $compressed);
```

- We create our directory path `$dir` by concatenating `'.mygit/objects/'` with the first two characters in `$hash`.
- If the path `$dir` didn't exist we make a new directory using `mkdir()`.
- We create our file path `$path` by concatenating `$dir` path with `/` and the rest of the file hash. Then we save the compressed content into our file.

7. Output and return the hash:
```php title="./index.php"
echo $hash . PHP_EOL;
return $hash;
```

- At the end, we output the calculated hash and return it.

Full code for this section:
```php title="./index.php"
function command_write_tree()
{
  if (!file_exists('.mygit/index')) {
    echo "Error: No index file found\n";
    return;
  }

  $entries = [];
  $index_content = file_get_contents('.mygit/index');

  if($index_content === ''){
    echo "Error: Index is empty\n";
    return;
  }

  $lines = explode("\n", trim($index_content));
  foreach ($lines as $line) {
    if (empty(trim($line)))
      continue;

    $parts = explode(" ", trim($line));
    if (count($parts) < 4)
      continue;

    $entries[] = $parts;
  }

  $tree_content = '';
  foreach ($entries as $entry) {
    $tree_content .= implode(" ", $entry) . "\n";
  }

  $header = "tree " . strlen($tree_content) . "\0";
  $full_content = $header . $tree_content;

  $hash = sha1($full_content);
  $compressed = gzcompress($full_content);

  $dir = ".mygit/objects/" . substr($hash, 0, 2);
  if (!is_dir($dir)) {
    mkdir($dir, 0777, true);
  }

  $path = $dir . "/" . substr($hash, 2);
  file_put_contents($path, $compressed);
  echo $hash . PHP_EOL;
  return $hash;
}
```