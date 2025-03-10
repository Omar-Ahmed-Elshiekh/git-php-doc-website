# ls-files command

The [ls-files command](https://git-scm.com/docs/git-ls-files) this command is used to list the files currently in the index file or the working tree, it is similar to `dir` or `ls` in the command line but for the index file.

### Command Implementation

1. Parse the Command Arguments:

In Git, ls-files function can take alot of flags but for simplicity in our implementation we'll create only one flag `-s` or `--stage`. This flag is used when we want to show staged contents' mode bits, object name and stage number in the output. but here we'll make output all index info.

```php title="./index.php"
$flag = $args[0] ?? null;
$index_path = './.mygit/index';
```

- We check if the user passed args (flags).
- We save the path of index file in `$index_path`.

2. Check if index file exists:

```php title="./index.php"
if (!file_exists($index_path)) {
  echo "Error: Index file not found. Have you added any files?\n";
  return;
}
```

- Check if the index file don't exists and output error message.

3. Read and validate index contents

```php title="./index.php"
$index_content = file_get_contents($index_path);

if (empty($index_content)) {
  return;
}
```

- We get the contents of index file and check if the index is empty.

4. Proccess and output index contents:

```php title="./index.php"
$index_entries = explode("\n", $index_content);
foreach ($index_entries as $entry) {
  $path = explode(" ", trim($entry), 4)[3];
  if ($flag === "--stage" || $flag === "-s") {
    echo $entry . "\n";
  } else if ($flag === null) {
    echo $path . "\n";
  } else {
    echo "Error: Undefined flag " . $flag . "\n";
    return;
  }
}
```

- We separate index contents into lines and store it in `$index_entries`.
- We separate each entry into four parts and we get the fourth element in the array which is the path.
- If the user entered the flag we output all the index entry data.
- Else if no flag provided we output just the path of the file.
- Else we output an error and return.

Full code for this section:
```php title="./index.php"
function command_ls_files($args)
{
  $flag = $args[0] ?? null;
  $index_path = './.mygit/index';

  if (!file_exists($index_path)) {
    echo "Error: Index file not found. Have you added any files?\n";
    return;
  }

  $index_content = file_get_contents($index_path);

  if (empty($index_content)) {
    return;
  }

  $index_entries = explode("\n", $index_content);
  foreach ($index_entries as $entry) {
    $path = explode(" ", trim($entry), 4)[3];
    if ($flag === "--stage" || $flag === "-s") {
      echo $entry . "\n";
    } else if ($flag === null) {
      echo $path . "\n";
    } else {
      echo "Error: Undefined flag " . $flag . "\n";
      return;
    }
  }
}
```