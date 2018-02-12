# history-service

## Overview

Stores your history on a central server. 

All you need is a port and a host to serve from, and your bash commands will be
sent to and retrieved from there.

![overviewimage](https://raw.githubusercontent.com/ianmiell/history-service/master/history-server.png)

Basic security is provided by a simple shared secret.

## Setup


Key for below:

```
PORTNUMBER - port you want to run on
HOSTNAME   - hostname you run the service on
YOURSECRET - your secret word for entry to service
```

- Put password for the service in `secret` file (on one line)

- Run: `chmod 400 secret` to make the file (relatively) secure

- Test it on the `localhost`:

Run, replacing `YOURSECRET` with your secret above.

```
$ ./history-server.sh PORTNUMBER &
printf 'YOURSECRET\ntest\n' | nc localhost PORTNUMBER
printf 'YOURSECRET\n\n' | nc localhost PORTNUMBER
kill %1
```

- Add `/path/to/history-service/server.sh` to cronjob to run as a service -
`run-one` takes care of duplicates

```
$ crontab -e
```

Then input the line (replacing the path):

```
* * * * * /path/to/history-service/history-server.sh
```

- Append the following to your ~/.bashrc file, replacing `YOURSECRET` with the
secret in the `secret` file and `HOSTNAME` with the host the service is running
on.

```
# history service
function history_service_send_last_command() { LAST=$(HISTTIMEFORMAT='' builtin history 1 | cut -c 8-); printf '%q' 'YOURSECRET\n'"${LAST}"'\n' | nc HOSTNAME PORTNUMBER; }
if [[ ${PROMPT_COMMAND} = '' ]]
then
	PROMPT_COMMAND="history_service_send_last_command"
else
	PROMPT_COMMAND="${PROMPT_COMMAND};history_service_send_last_command"
fi
alias historyservice="printf '%q' 'YOURSECRET\n\n' | nc HOSTNAME PORTNUMBER'
```

The security level of this is sufficent to stop casual users from abusing your
file system or getting access (assuming your secret is strong enough and kept
safe), but is not enough to stop a determined attacker from doing damage.
Use at your own risk.

## Requirements

Requires:

- `bash` v4+

Check your version with:

```
echo ${BASH_VERSION[0]}
```

If you are on a Mac, you may want to `brew install bash` to get a later version.
The default one that ships is a 3.x version (!?).

- `socat`

http://www.dest-unreach.org/socat/

Available on most package managers.

## Bugs / TODOs

- Multi-line commands not well-handled
- Unique the commands?
- Record the date and source of the command?
- Create a history db?
