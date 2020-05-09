# Getting a proper shell!

If you got a crappy shell, you can fully up it to a full bash terminal.

Call bash via python or python3:

```text
python -c 'import pty; pty.spawn("/bin/bash")'
python3 -c 'import pty; pty.spawn("/bin/bash")'
```

background it with ctrl + z

run `stty raw -echo` in your machine

foreground netcat with `fg` and then type reset.

If it asks for your terminal, go back to you bash, type `echo $TERM`, copy the output and paste in the netcat.

Congrats, you have a full bash via netcat.

