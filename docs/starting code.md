# Starting Code

First, we create the main file that we are going to work on which is index.php 

We will start this by making a variable command this will store the command entered by the user and an array commands which will contain all the commands in our projects. 

```php title="./index.php" showLineNumbers
<?php

$command = $argv[1] ?? null;
$commands = ['init', 'add', 'commit', 'log', 'hash_object', 'cat_file', 'write_tree', 'ls_tree', 'ls_files'];
```

The line `$command = $argv[1] ?? null;` will take the command from user which will be at index 1 in [`$argv`](https://www.php.net/manual/en/reserved.variables.argv.php) and store in in `$command`, if there is no args it will be null.

The line `$commands = ['init', ...` is the set of commands that we have in our project that we will use to validate the user input.

Then we have to check that the command entered by the user is valid.

```php title="./index.php" showLineNumbers
if (!$command || !in_array($command, $commands)) {
  echo "Error: Usage ./index.php <$command>\n";
  echo "Available commands: init, add, commit, log, hash_object, cat_file, write_tree, ls_tree, ls_files\n";
  exit(1);
}
```

This code check if the `$command` is empty or the `$command` is not found in `$commands` array which means there was an error in the user input and echo error massages to user and exit.

If the if condition did not excute then there were no errors and we take input from user and pass it to [`call_user_func()`](https://www.php.net/manual/en/function.call-user-func.php).

```php title="./index.php"
call_user_func("command_$command", array_slice($argv, 2));
```

First argument is the name of our command and second argument is the rest of the args entered by user. 

Full code for this section: 
```php title="./index.php" showLineNumbers
<?php

$command = $argv[1] ?? null;
$commands = ['init', 'add', 'commit', 'log', 'hash_object', 'cat_file', 'write_tree', 'ls_tree', 'ls_files'];

if (!$command || !in_array($command, $commands)) {
  echo "Error: Usage ./index.php <$command>\n";
  echo "Available commands: init, add, commit, log, hash_object, cat_file, write_tree, ls_tree\n";
  exit(1);
}

call_user_func("command_$command", array_slice($argv, 2));
```