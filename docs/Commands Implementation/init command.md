# init command

The [init command](https://git-scm.com/docs/git-init) is a foundational step in creating a version-controlled project, and it is used to create a **repository**.

:::info
**Repository** is a collection of refs together with an object database containing all objects which are reachable from the refs, possibly accompanied by meta data from one or more porcelains. A repository can share an object database with other repositories via alternates mechanism.             
-kernal.org
:::

In real Git it creates the `.git` directory in the root of your repo, but in this project, we’ll use `.mygit` as the repository directory for clarity and to distinguish it from real Git repositories.

We start by making function `command_init()` then:

1. Check if a directory with the name `.mygit` already exists:
```php title="index.php"
if (is_dir('.mygit')) {
  echo "Repository already initialized.\n";
  return;
}
```
  - If it does that means we have already initialized a repository and we output `Repository already initialized.` and return. 

2. Create our repository structure:
```php title="index.php"
mkdir('.mygit');
mkdir('.mygit/objects');
mkdir('.mygit/refs');
file_put_contents('.mygit/HEAD', "ref: refs/heads/main\n");
file_put_contents('.mygit/index', '');
```
  - `.mygit`: As we said earlier it is the root directory of our repo. It will store all the metadata needed for version control.
  - `.mygit/objects`: This directory will store all our objects (blobs, trees, and commits) "We will explain each one of them" created in our repo, and each object will be stored with a unique hash.
  - `.mygit/refs`: This directory will contain references, such as branch heads. Branches in Git are pointers to specific commits, and this is where those pointers are stored. 

3. Create the HEAD file:
```php title="./index.php"
file_put_contents('.mygit/HEAD', "ref: refs/heads/main\n");
```
- `HEAD`: It determines the current branch the repo is pointing to, and we initialize it with the main branch `refs/heads/main` "which we will create later".

4. Create the index file:

```php title="./index.php"
file_put_contents('.mygit/index', '');
```

- `index`: The index is a file that stores a list of file names, along with file metadata. (more on that later)

Although we won't implement the branching functionality on this project and will work only on the main branch still `.mygit/refs` and `HEAD` are essential parts of Git's architecture.

At the end, we output a success message `Initialized MyGit repository.` to the user.

Now try to run the code using: 
```
php ./index.php init
```

And the .mygit directory structure should look like this:
```
.mygit/
├── objects/
├── refs/
│   └── heads/
└── HEAD
└── index
```

```text title="run"
php ./index.php init
```

Full code for this section:
```php title="index.php" showLineNumbers
function command_init()
{
  if (is_dir('.mygit')) {
    echo "Repository already initialized.\n";
    return;
 }

  mkdir('.mygit');
  mkdir('.mygit/objects');
  mkdir('.mygit/refs');
  file_put_contents('.mygit/HEAD', "ref: refs/heads/main\n");
  file_put_contents('.mygit/index', '');
  echo "Initialized MyGit repository.\n";
}
```