# add command

The [add command](https://git-scm.com/docs/git-add) is used to add file contents to the index. This command updates the index using the current content found in the working tree, to prepare the content staged for the next commit.

In Git, the index, often referred to as the staging area, is a crucial component of the version control process. It serves as a middle layer between your working directory and the repository, enabling you to organize and review changes before committing them. The index is a binary file that stores a sorted list of file names, along with file metadata and pointers to the object database but in this project we'll store the data in a normal text format for simplicity.

## Command Implementation

First, we define the function `command_add($args)`.

1. Check if index file exists:

```php title="./index.php"
if (!file_exists(".mygit/index")) {
  file_put_contents(".mygit/index", '');
}
```

  - If the index does not exist we create an empty index file.

2. Check if user input:

```php title="./index.php"
if (empty($args)) {
  echo "Error: No paths specified for adding\n";
  return;
}
```

  - We check if `$args` is empty that means the user didn't provide any input path so we print an error and return.

3. Get index current content:

```php title="./index.php"
$index_entries = [];
$index_contents = file_get_contents('.mygit/index');
if ($index_contents) {
  $index_entries = array_filter(explode("\n", $index_contents));
}
```

  - First, we create `$index_entries` array which will contain each line of index contents.
  - We get the contents of index file and save it in `$index_contents`.
  - We check if `$index_contents` has a value, if it does we split the `$index_contents` string into an array and store it in `$index_entries`.

4. Create index mapping:

```php title="./index.php"
$index = [];

foreach ($index_entries as $entry) {
  $parts = explode(" ", $entry);
  if (count($parts) >= 4) {
    $path = $parts[3];
    $index[$path] = $entry;
  }
}
``` 

 - For each entry, each entry will look like this ex:`100644 blob 595a9f9dcdc6e92bd7b88be3673dd1df22ceca67 index.php`, we split it into 4 parts,
 then we validate that parts have 4 elements, and we map each path which is at index 3 to the entire entry.

5. Process provided paths:

```php title="./index.php"
foreach ($args as $path) {
  if (!file_exists($path)) {
    echo "Error: '$path' does not exist\n";
    continue;
  }

  if (is_file($path)) {
    add_file($path, $index);
  } else if (is_dir($path)) {
    add_dir($path, $index);
  }
}
```

- For each path given in $args, the function checks if the file/directory exists. If not, it prints an error message and skips that entry.
- If the path points to a file, add_file() is called; if it’s a directory, add_dir() is called (which we'll explain in a minute).

6. Store new index contents:

```php title="./index.php"
ksort($index);
$new_content = implode("\n", $index);
if ($new_content) {
  $new_content .= "\n";
}

file_put_contents(".mygit/index", $new_content);
```

- In Git, the data stored in index file are sorted so we use [`ksort`](https://www.php.net/manual/en/function.ksort.php) built-in function to sort index contents before we store them.
- Then we convert the `$index` array into a string separated with `\n` and check if the `$new_content` is not empty we concatenate a `\n` to it for future use.
- Then we store the new content into the index file.

## Helper functions

7. add_file() function:

```php title="./index"
function add_file($path, &$index)
{
  if (str_starts_with($path, './mygit'))
    return;
  $hash = command_hash_object([$path, 1]);
  $mode = '100644';
  $type = 'blob';
  $entry = "$mode $type $hash $path";
  $index[$path] = $entry;
}
```
- We pass the `$path` of a file and reference to index array `&$index` so we make changes to current index not just to take a copy from it.
- We ingonre the MyGit directory to prevent tracking repo metadata.
- We compute the hash of the object using the `command_hash_object([$path, 1])` function we created earlier by giving it the `$path` and 1 to write enable.
- Then save the entry in the format `<mode> <type> <hash> <path>`.

8. add_dir() function:

```php title="./index.php"
function add_dir($dir, &$index)
{
  $items = scandir($dir);

  foreach ($items as $item) {
    if ($item === '.' || $item === '..' || $item === '.mygit') {
      continue;
    }

    $path = $dir === '.' ? $item : "$dir/$item";

    if (is_file($path)) {
      add_file($path, $index);
    } else if (is_dir($path)) {
      add_dir($path, $index);
    }
  }
}
```

- Uses scandir($dir) to retrieve all files and subdirectories.
- Ignores . (current directory), .. (parent directory), .mygit, and .git directories.
- Calls add_file() for files.
- Calls itself (add_dir()) recursively for subdirectories.

Full code for this section:
```php title="./index.php"
function command_add($args)
{
  if (!file_exists(".mygit/index")) {
    file_put_contents(".mygit/index", '');
  }

  if (empty($args)) {
    echo "Error: No paths specified for adding\n";
    return;
  }

  $index_entries = [];
  $index_contents = file_get_contents('.mygit/index');
  if ($index_contents) {
    $index_entries = array_filter(explode("\n", $index_contents));
  }

  $index = [];

  foreach ($index_entries as $entry) {
    $parts = explode(" ", $entry);
    if (count($parts) >= 4) {
      $path = $parts[3];
      $index[$path] = $entry;
    }
  }

  foreach ($args as $path) {
    if (!file_exists($path)) {
      echo "Error: '$path' does not exist\n";
      continue;
    }

    if (is_file($path)) {
      add_file($path, $index);
    } else if (is_dir($path)) {
      add_dir($path, $index);
    }
  }

  ksort($index);
  $new_content = implode("\n", $index);
  if ($new_content) {
    $new_content .= "\n";
  }

  file_put_contents(".mygit/index", $new_content);

}

function add_file($path, &$index)
{
  if (str_starts_with($path, './mygit'))
    return;
  $hash = command_hash_object([$path, 1]);
  $mode = '100644';
  $type = 'blob';
  $entry = "$mode $type $hash $path";
  $index[$path] = $entry;
}

function add_dir($dir, &$index)
{
  $items = scandir($dir);

  foreach ($items as $item) {
    if ($item === '.' || $item === '..' || $item === '.mygit') {
      continue;
    }

    $path = $dir === '.' ? $item : "$dir/$item";

    if (is_file($path)) {
      add_file($path, $index);
    } else if (is_dir($path)) {
      add_dir($path, $index);
    }
  }
}
```