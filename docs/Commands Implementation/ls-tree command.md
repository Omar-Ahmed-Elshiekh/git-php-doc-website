# ls-tree command

The [ls-tree command](https://git-scm.com/docs/git-ls-tree) is used to list the contents of a given tree object showing their metadata such as mode, type, hash, and name. This command is useful for inspecting a specific version of a repository’s directory structure.

### Command Implementation

1. Parse the Command Arguments:

In Git, ls-tree function can take alot of flags but for simplicity in our implementation we'll create only one flag `--name-only`. This flag is used when we want to list only filenames (instead of the "long" output), one per line.

```php title="./index.php"
$hash = $args[0] ?? null;
$flag = false;

if (isset($args[0]) && $args[0] === "--name-only") {
  if (count($args) < 2) {
    echo "Error: Hash required.\n";
    return;
  }
  $flag = true;
  $hash = $args[1];
}
```

- Extract the first argument which is the tree object's hash.
- Checks if the --name-only flag is passed:
    - If --name-only is found, it requires an additional argument (the hash).
    - If no hash is provided after the flag, it prints an error and exits.
    - Otherwise, it sets $flag = true, meaning the output will include only filenames.

2. Locate and retrieve the tree object:

```php title="./index.php"
$dir = '.mygit/objects/' . substr($hash, 0, 2);
$file = $dir . '/' . substr($hash, 2);

if (!file_exists($file)) {
  echo "Error: Tree object not found.\n";
  return;
}

$compressed = file_get_contents($file);
$data = gzuncompress($compressed);
```

- We create our directory path `$dir` by concatenating `'.mygit/objects/'` with the first two characters in `$hash`.
- We create our file path `$file` by concatenating `$dir` path with `/` and the rest of the file hash.
- If the file doesn't exist output an error and return.
- Then we get the compressed file contents from `$file` and store the uncompressed data in `$data`.

3. Validate tree object format:

```php title="./index.php"
$parts = explode("\0", $data, 2);
if (count($parts) < 2) {
  echo "Error: Invalid object format.\n";
  return;
}

[$header, $content] = $parts;
if (!str_starts_with($header, 'tree ')) {
  echo "Error: Not a tree object.\n";
  return;
}
```

- Split the uncompressed data into two parts header and content.
- Validate that `$parts` has the two parts and output an error and return if not.
- Check if our header starts with `'tree '` (which means it is a tree object) and output an error and return if not.

4. Parse and display tree entries:

```php title="./index.php"
$lines = explode("\n", trim($content));

foreach ($lines as $line) {
  if (empty($line))
    continue;

  $parts = explode(" ", trim($line), 4);
  if (count($parts) !== 4) {
    continue;
  }

  [$mode, $type, $hash, $name] = $parts;

  if ($flag) {
    echo $name . "\n";
  } else {
    echo "$mode $type $hash $name\n";
  }
}
```

- Split the tree content into lines, each representing a file entry.
- Iterates through each line:
    - Skips empty lines.
    - Splits the line into four parts: mode, type, hash, and name.
    - If --name-only is set, only prints filenames.
    - Otherwise, print the full metadata: mode, type, hash, name.

Full code for this section:
```php title="./index.php"
function command_ls_tree($args)
{
  $hash = $args[0] ?? null;
  $flag = false;

  if (isset($args[0]) && $args[0] === "--name-only") {
    if (count($args) < 2) {
      echo "Error: Hash required.\n";
      return;
    }
    $flag = true;
    $hash = $args[1];
  }

  $dir = '.mygit/objects/' . substr($hash, 0, 2);
  $file = $dir . '/' . substr($hash, 2);

  if (!file_exists($file)) {
    echo "Error: Tree object not found.\n";
    return;
  }

  $compressed = file_get_contents($file);
  $data = gzuncompress($compressed);
  $parts = explode("\0", $data, 2);
  if (count($parts) < 2) {
    echo "Error: Invalid object format.\n";
    return;
  }

  [$header, $content] = $parts;
  if (!str_starts_with($header, 'tree ')) {
    echo "Error: Not a tree object.\n";
    return;
  }

  $lines = explode("\n", trim($content));

  foreach ($lines as $line) {
    if (empty($line))
      continue;

    $parts = explode(" ", trim($line), 4);
    if (count($parts) !== 4) {
      continue;
    }

    [$mode, $type, $hash, $name] = $parts;

    if ($flag) {
      echo $name . "\n";
    } else {
      echo "$mode $type $hash $name\n";
    }
  }
}
```