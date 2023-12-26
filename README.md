This guide will walk you through setting up an iOS Shortcut that interacts with the `iSH` app on an iOS device. It utilizes SSH to run a script within a `screen` session that prints a clickable hyperlink in the terminal. This hyperlink can trigger another iOS Shortcut, passing data back to it.

## Step 1: Install Packages in `iSH`

Open `iSH` and run:

```bash
apk update
apk add screen openssh jq
```

## Step 2: Configure SSH

SSH on iOS must run on a nonstandard port, I didn't really look into why. I used port 2200.

1. Edit the SSH config file:

```bash
nano /etc/ssh/sshd_config
```

2. Add or modify the following lines:

```
Port 2200
PermitRootLogin yes
```

3. Modify .profile to start screen and the SSH server:

```bash
/usr/sbin/sshd &
screen -S main
```

then reload iSH or run
```bash
/usr/sbin/sshd
screen -S main
```

4. Set a password for your root user:

It would probably be much more secure to make a user for this, but this is just my proof of functionality. 

```bash
passwd
```

## Step 3: Create the Link Printing Script

1. Create `print_link.sh`:

```bash
nano print_link.sh
```

2. Insert the script:

```bash
#!/bin/sh

message="Sent Back to Shortcuts"
encoded_message=$(jq -rn --arg msg "$message" '$msg|@uri')
url="shortcuts://run-shortcut?name=Cheese&input=${encoded_message}&text=[text]"
text="Click Me"

# Print the hyperlink
printf '\eP\e]8;;%s\a%s\eP\e]8;;\a\e\\\n' "$url" "$text"
```

3. Save, exit (`Ctrl` + `X`, then `Y`, then `Enter`), and make the script executable:

```bash
chmod +x print_link.sh
```

## Step 4: Set Up the iOS Shortcut

1. Open the Shortcuts app and create a new shortcut.

2. Add actions:
   - **Recieve Text**: from 'nowhere'
   - **Text**: This holds the SSH command.
   - **Run Script Over SSH**: Configure with `iSH` SSH details.

3. Fill in the details:

   - **Host**: `127.0.0.1`
   - **Port**: `2200`
   - **User**: `root`
   - **Password**: Your iSH password
   - **Script**: The path to `print_link.sh`. '~/print_link.sh' for me here.

4. Add an 'If' action to check for input.

5. If input is received, show an alert; otherwise, run the SSH script.

## Step 5: Running the Shortcut

Execute the shortcut. The hyperlink should appear in `iSH` and be clickable.

## Step 6: Configure the Receiver Shortcut

I used  a Shortcut named "Cheese" that both sends the original command and processes the input from the hyperlink click.


This should be able to open and pass variables back to any iOS app that supports URI. 
