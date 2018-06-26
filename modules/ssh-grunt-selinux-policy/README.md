**Note**: This public repo contains the documentation for the private GitHub repo <https://github.com/gruntwork-io/module-security>.
We publish the documentation publicly so it turns up in online searches, but to see the source code, you must be a Gruntwork customer.
If you're already a Gruntwork customer, the original source for this file is at: <https://github.com/gruntwork-io/module-security/blob/master/modules/ssh-grunt-selinux-policy/README.md>.
If you're not a customer, contact us at <info@gruntwork.io> or <http://www.gruntwork.io> for info on how to get access!

# SSH Grunt SELinux Policy

This module installs a SELinux Local Policy Module that is necessary to make [ssh-grunt](/modules/ssh-grunt) work on
systems with SELinux, such as CentOS. 

The reason we need a policy is that `ssh-grunt` uses is executed on each attempted SSH login by the
`AuthorizedKeysCommand`, and as part of that process, `ssh-grunt` makes an HTTP call to the local EC2 Instance Metadata
(to access an IAM role), and this call is blocked by default by SELinux. Installing this module gives permissions to
`sshd` to successfully make this HTTP call.




## Install

The easiest way to use this module is via the [Gruntwork
Installer](https://github.com/gruntwork-io/gruntwork-installer) (make sure to replace `<VERSION>` below with the latest
version from the [releases page](https://github.com/gruntwork-io/module-security-public/releases)):

```
gruntwork-install --module-name ssh-grunt-selinux-policy --repo https://github.com/gruntwork-io/module-security --tag <VERSION>
```



## How this policy was created

*This section is primarily for the module maintainers*.

If you install `ssh-grunt` on SELinux and try to SSH to the server, you'll see the following errors in
`/var/log/audit/audit.log`:


```
type=AVC msg=audit(1519835859.599:55): avc:  denied  { name_connect } for  pid=1080 comm="ssh-grunt" dest=80 scontext=system_u:system_r:sshd_t:s0-s0:c0.c1023 tcontext=system_u:object_r:http_port_t:s0 tclass=tcp_socket
type=SYSCALL msg=audit(1519835859.599:55): arch=c000003e syscall=42 success=no exit=-13 a0=3 a1=c4200bd42c a2=10 a3=0 items=0 ppid=1079 pid=1080 auid=4294967295 uid=0 gid=0 euid=0 suid=0 fsuid=0 egid=0 sgid=0 fsgid=0 tty=(none) ses=4294967295 comm="ssh-grunt" exe="/usr/local/bin/ssh-grunt" subj=system_u:system_r:sshd_t:s0-s0:c0.c1023 key=(null)
```

These error logs are a bit hard to read, so you may want to install `setroubleshoot` to get human-friendly error 
messages:

```
$ sudo yum install -y setroubleshoot
$ sealert -a /var/log/audit/audit.log

SELinux is preventing ssh-grunt from name_connect access on the tcp_socket port 80.

*****  Plugin catchall (100. confidence) suggests   **************************

If you believe that ssh-grunt should be allowed name_connect access on the port 80 tcp_socket by default.
Then you should report this as a bug.
You can generate a local policy module to allow this access.
Do
allow this access for now by executing:
# grep ssh-grunt /var/log/audit/audit.log | audit2allow -M mypol
# semodule -i mypol.pp
```

Following the instructions above, we created the Local Policy Module files by running:

```
$ grep ssh-grunt /var/log/audit/audit.log | audit2allow -M ssh-grunt
```

This creates a Local Module Policy consisting of two files: `ssh-grunt.pp` and `ssh-grunt.te`, which we have copied into
this module. These files can be installed by running:

```
$ sudo semodule -i ssh-grunt.pp
```