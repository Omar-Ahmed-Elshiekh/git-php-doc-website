# log command

The [log command](https://git-scm.com/docs/git-log) is used to view the commit history of a repository. It displays:

- The commit hash (SHA-1 identifier)
- The author of the commit
- The commit timestamp
- The commit message

### Command Implementation

1. Check if `.mygit/HEAD` exists:

```php title="./index.php"
if (!file_exists(".mygit/HEAD")) {
  echo "Error: No commits yet\n";
  return;
}
```

- Ensures that the repository has at least one commit.
- If `.mygit/HEAD` doesn’t exist, no commits have been made, so it prints an error and exits.

2. Determine the latest commit hash:

```php title="./index.php"
$head_content = trim(file_get_contents(".mygit/HEAD"));
$curr_hash = null;

if (str_starts_with($head_content, "ref: ")) {
  $ref = substr($head_content, 5);
  if (!file_exists(".mygit/" . $ref)) {
    echo "Error: No commits yet\n";
    return;
  }
  $curr_hash = trim(file_get_contents(".mygit/" . $ref));
} else {
  $curr_hash = $head_content;
}
```

- Reads .mygit/HEAD, which stores either:
    - A reference (`ref: refs/heads/main`), pointing to a branch’s latest commit.
    - A direct commit hash (in a detached HEAD state).
- If it’s a reference, we retrieve the latest commit hash from `.mygit/refs/heads/main`.
- Otherwise, we assume that `.mygit/HEAD` contains a commit hash.

3. Iterate through the commit history:

```php title="./index.php"
while ($curr_hash) {
```

- The loop iterates over commit objects, starting from `$curr_hash` (latest commit) and following parent commits.

4. Retrieve and uncompress the commit object:

```php title="./index.php"
$dir = ".mygit/objects/" . substr($curr_hash, 0, 2);
$path = $dir . '/' . substr($curr_hash, 2);

if (!file_exists($path)) {
  echo "Error: Commit object not found\n";
  break;
}

$compressed = file_get_contents($path);
$data = gzuncompress($compressed);
$parts = explode("\0", $data, 2);

if (count($parts) < 2) {
  echo "Error: Invalid commit object format\n";
  break;
}
```

- we construct the path to the commit object by using the first two characters of the hash as the directory name and the rest as file path.
- Reads and uncompresses the commit object.
- Splits the content at the `\0` null byte (used to separate the header and commit content).
- If the commit object format is invalid, it prints an error and exits.

5. Extract commit metadata:

    1.  Extract the content:

    ```php title="./index.php"
    $content = $parts[1];
    $lines = explode("\n", $content);
    ```
    - `$parts[1]` contains everything after the `\0`, which the actual commit metadata and message.
    - Then we split the content into lines.

    2. Initialize variables:
    ```php title="./index.php"
    $commit_data = [];
    $message = "";
    $reading_message = false;
    ```
    - `$commit_data` will store key-value pairs of commit metadata (tree, parent, author).
    - `$message` will store the commit message.
    - `$reading_message` is a flag to track when we start reading the message.

    3. Process each line:
    ```php title="./index.php"
    foreach ($lines as $line) {
      if (empty(trim($line))) continue;
    ```
    - We skip empty lines to avoid unnecessary processing.

    4. Handle the commit message:
    ```php title="./index.php"
    if (str_starts_with($line, "message ")) {
      $message = substr($line, 8);
      $reading_message = true;
    } else if ($reading_message) {
      $message .= "\n" . $line;
    } else {
      $parts = explode(" ", $line, 2);
      if (count($parts) == 2) {
        $commit_data[$parts[0]] = $parts[1];
      }
    }
    ```
    - If the line starts with "message ", it marks the beginning of the commit message.
    - We use `substr($line, 8)` to remove "message " and stores the actual message.
    - If `$reading_message` is already true, additional lines are appended to `$message` (we use it to handle multi-line commit messages).
    - Else if a line isn’t part of the message, we store it as a key-value pair after separating it into parts.

    Example of what's stored in our variables after processing:
    ```php title="example"
    $commit_data = [
      "tree" => "a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s",
      "parent" => "7a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6q7r8s",
      "author" => "John Doe john@example.com 1707301245"
    ];

    $message = "Fixed a major bug\nthat caused a crash.";
    ```
This part code:
```php title="./index.php"
$content = $parts[1];
$lines = explode("\n", $content);

$commit_data = [];
$message = "";
$reading_message = false;

foreach ($lines as $line) {
  if (empty(trim($line))) continue;

  if (str_starts_with($line, "message ")) {
    $message = substr($line, 8);
    $reading_message = true;
  } else if ($reading_message) {
    $message .= "\n" . $line;
  } else {
    $parts = explode(" ", $line, 2);
    if (count($parts) == 2) {
      $commit_data[$parts[0]] = $parts[1];
    }
  }
}
```

6. Print commit data:

```php title="./index.php"
echo "\033[33mcommit " . $curr_hash . "\033[0m\n";

if (isset($commit_data["author"])) {
  $author_data = explode(" ", $commit_data["author"]);
  $timestamp = end($author_data);
  $author_name = implode(" ", array_slice($author_data, 0, -1));
  $date = date("Y-m-d H:i:s", (int) $timestamp) . "\n";
}
echo "\033[36mAuthor: " . $author_name . "\033[0m\n";
echo "Date: " . $date;
echo "\n      " . $message . "\n\n";
```

- Prints the commit hash in yellow (`\033[33m`).
- We extracts the author's name and timestamp.
- Then we convert the timestamp into a human-readable format (`YYYY-MM-DD HH:MM:SS`).
- Prints the author’s name in cyan (`\033[36m`).
- Prints the commit message.

7. Move to the parent commit:

```php title="./index.php"
$curr_hash = $commit_data["parent"] ?? null;
```

- Sets `$curr_hash` to the parent commit hash.
- If no parent exists, it stops our while loop.

Full code for this section:
```php title="./index.php"
function command_log()
{
  if (!file_exists(".mygit/HEAD")) {
    echo "Error: No commits yet\n";
    return;
  }

  $head_content = trim(file_get_contents(".mygit/HEAD"));
  $curr_hash = null;

  if (str_starts_with($head_content, "ref: ")) {
    $ref = substr($head_content, 5);
    if (!file_exists(".mygit/" . $ref)) {
      echo "Error: No commits yet\n";
      return;
    }
    $curr_hash = trim(file_get_contents(".mygit/" . $ref));
  } else {
    $curr_hash = $head_content;
  }

  while ($curr_hash) {
    $dir = ".mygit/objects/" . substr($curr_hash, 0, 2);
    $path = $dir . '/' . substr($curr_hash, 2);

    if (!file_exists($path)) {
      echo "Error: Commit object not found\n";
      break;
    }

    $compressed = file_get_contents($path);
    $data = gzuncompress($compressed);
    $parts = explode("\0", $data, 2);

    if (count($parts) < 2) {
      echo "Error: Invalid commit object format\n";
      break;
    }

    $content = $parts[1];
    $lines = explode("\n", $content);

    $commit_data = [];
    $message = "";
    $reading_message = false;

    foreach ($lines as $line) {
      if (empty(trim($line)))
        continue;

      if (str_starts_with($line, "message ")) {
        $message = substr($line, 8);
        $reading_message = true;
      } else if ($reading_message) {
        $message .= "\n" . $line;
      } else {
        $parts = explode(" ", $line, 2);
        if (count($parts) == 2) {
          $commit_data[$parts[0]] = $parts[1];
        }
      }

    }

    echo "\033[33mcommit " . $curr_hash . "\033[0m\n";

    if (isset($commit_data["author"])) {
      $author_data = explode(" ", $commit_data["author"]);
      $timestamp = end($author_data);
      $author_name = implode(" ", array_slice($author_data, 0, -1));
      $date = date("Y-m-d H:i:s", (int) $timestamp) . "\n";
    }
    echo "\033[36mAuthor: " . $author_name . "\033[0m\n";
    echo "Date: " . $date;
    echo "\n      " . $message . "\n\n";

    $curr_hash = $commit_data["parent"] ?? null;
  }

}
```