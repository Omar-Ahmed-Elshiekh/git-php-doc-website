# commit command

The [commit command](https://git-scm.com/docs/git-commit) is used to record changes to the repository by creating a new commit object containing the current contents of the index and the given log message describing the changes.

Each commit object includes:
- A snapshot (tree object) of the current state of the repository.
- A reference to the parent commit (if any).
- Metadata (author, timestamp, commit message).

### Command implementation

1. Validate input arguments:

```php title="./index.php"
if (!isset($args[0]) || $args[0] !== '-m' || !isset($args[1]) || trim($args[1]) === "") {
  echo "Error: Please provide a commit message using -m flag\n";
  return;
}

$msg = $args[1];
```

- First, we validate that the `-m` flag (which stands for message) and the commit message are provided and not empty.
- Then we store our message in `$msg`.

2. Create a tree object (snapshot of the working directory):

```php title="./index.php"
  $tree_hash = command_write_tree();
  if (!$tree_hash) {
    echo "Error: Nothing to commit\n";
    return;
  }
```

- We create a new tree object from the current index using the `command_write_tree()` function that we created earlier.
- Then we validate that it is not empty.

3. Determine the parent commit (if any):

```php title="./index.php"
$parent_hash = null;
if (file_exists(".mygit/HEAD")) {
  $head_content = trim(file_get_contents(".mygit/HEAD"));
  if(str_starts_with($head_content,"ref: ")){
    $ref = substr($head_content, 5);
    if (file_exists(".mygit/" . $ref)) {
      $parent_hash = trim(file_get_contents(".mygit/" . $ref));
    }
  }
}
```

- Checks if `.mygit/HEAD` exists (which stores the current branch reference).
- If `.mygit/HEAD` contains a reference (`ref: refs/heads/main`), it extracts the branch name (main) and reads the latest commit hash from `.mygit/refs/heads/main`.
- If this file exists, it stores the commit hash as `$parent_hash`.

4. Construct commit object data:

```php title="./index.php"
$author = "John Doe <john@example.com>";
$timestamp = time();

$commit_content = "tree " . $tree_hash . "\n";
if ($parent_hash) {
  $commit_content .= "parent " . $parent_hash . "\n";
}
$commit_content .= "author " . $author . " " . $timestamp . "\n";
$commit_content .= "committer " . $author . " " . $timestamp . "\n";
$commit_content .= "message " . $msg . "\n";
```

- Defines the commit author, here we hardcoded it for simplicity.
- Get the current timestamp (commit time).
- Start constructing the commit content by concatenating:
    - Tree hash.
    - Parent commit hash (if any).
    - Author with timestamp.
    - Committer with timestamp.
    - Commit message.

:::note
Committer and Author can be different
:::

- The final commit format will be like this:
```text title="example"
tree 9daeafb9864cf43055ae93beb0afd6c7d144bfa4
parent e69de29bb2d1d6434b8b29ae775ad8c2e48c5391
author John Doe <john@example.com> 1707395265
committer John Doe <john@example.com> 1707395265
```

:::note
In real Git, you cannot create multiple commits with the same index content because Git verifies whether the working directory has changed since the last commit. This is done using metadata stored in the index file.
However, in this simplified implementation, we only check if the tree hash is empty, not if it has already been committed. And because we use time() for the commit timestamp, a new commit is always created, even if no changes were made.
:::

5. Add a commit object header:

```php title="./index.php"
$header = "commit " . strlen($commit_content) . "\0";
$full_content = $header . $commit_content;
```

- As we have done before, we prepare commit object format.

6. Compute the commit hash:

```php title="./index.php"
$hash = sha1($full_content);
$compressed = gzcompress($full_content);
```

- Compute the commit hash and compress its content.

7. Store the commit object:

```php title="./index.php"
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

8. Update the branch reference (HEAD):

```php title="./index.php"
if (!is_dir(".mygit/refs/heads")) {
  mkdir(".mygit/refs/heads", 0777, true);
}

file_put_contents('.mygit/refs/heads/main', $hash);

echo "Created commit " . $hash . "\n";
```

- Ensures that `.mygit/refs/heads/` exists.
- Updates `.mygit/refs/heads/main` to point to the new commit hash, marking it as the latest commit.
- Print the commit hash to the user.

Full code for this section:
```php title="./index.php"
function command_commit($args)
{
  if (!isset($args[0]) || $args[0] !== '-m' || !isset($args[1]) || trim($args[1]) === "") {
    echo "Error: Please provide a commit message using -m flag\n";
    return;
  }

  $msg = $args[1];

  $tree_hash = command_write_tree();
  if (!$tree_hash) {
    echo "Error: Nothing to commit\n";
    return;
  }

  $parent_hash = null;
  if (file_exists(".mygit/HEAD")) {
    $head_content = trim(file_get_contents(".mygit/HEAD"));
    if(str_starts_with($head_content,"ref: ")){
      $ref = substr($head_content, 5);
      if (file_exists(".mygit/" . $ref)) {
        $parent_hash = trim(file_get_contents(".mygit/" . $ref));
      }
    }
  }

  $author = "John Doe <john@example.com>";
  $timestamp = time();

  $commit_content = "tree " . $tree_hash . "\n";
  if ($parent_hash) {
    $commit_content .= "parent " . $parent_hash . "\n";
  }
  $commit_content .= "author " . $author . " " . $timestamp . "\n";
  $commit_content .= "committer " . $author . " " . $timestamp . "\n";
  $commit_content .= "message " . $msg . "\n";

  $header = "commit " . strlen($commit_content) . "\0";
  $full_content = $header . $commit_content;

  $hash = sha1($full_content);
  $compressed = gzcompress($full_content);

  $dir = ".mygit/objects/" . substr($hash, 0, 2);
  if (!is_dir($dir)) {
    mkdir($dir, 0777, true);
  }

  $path = $dir . "/" . substr($hash, 2);
  file_put_contents($path, $compressed);

  if (!is_dir(".mygit/refs/heads")) {
    mkdir(".mygit/refs/heads", 0777, true);
  }

  file_put_contents('.mygit/refs/heads/main', $hash);

  echo "Created commit " . $hash . "\n";
}
```