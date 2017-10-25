Agent Configuration: Linux
==========================

In order to use an OpenPGP smart card for SSH, the ``SSH_AUTH_SOCK``
environment variable needs to point at a GnuPG agent (``gpg-agent``) socket.
Getting this right can be tricky.

Debian 9 (stretch) and Xfce
---------------------------

If you use Xfce on Debian stretch or newer (including buster), getting this
working is actually quite straightforward.

TL;DR (quick setup)::

  xfconf-query -c xfce4-session -p /startup/ssh-agent/enabled -n -t bool -s false
  echo use-standard-socket >> ~/.gnupg/gpg-agent.conf
  echo enable-ssh-support >> ~/.gnupg/gpg-agent.conf

Xfce starts both ``gpg-agent`` and ``ssh-agent`` instances for you at login,
but you want to avoid the standalone ``ssh-agent`` and instead configure just
``gpg-agent`` for SSH. In fact, the ``gpg-agent`` is likely started for you as
a systemd user service: you probably want to keep that setup.

To make sure Xfce doesn't start ``ssh-agent`` for you, you need to disable it
in your settings. Unfortunately this isn't presented anywhere in the GUI that I
can find, but can be easily `disabled on the command-line
<http://docs.xfce.org/xfce/xfce4-session/advanced#ssh_and_gpg_agents>`::

  xfconf-query -c xfce4-session -p /startup/ssh-agent/enabled -n -t bool -s false

Then you need to tell ``gpg-agent`` to always enable its SSH support, and
ideally for it to use a standard (stable) socket path rather than something
random in ``/tmp``. The latter is particularly useful if you need to restart
``gpg-agent`` for any reason, as the environment variables get baked into your
desktop session, and is the default in GnuPG 2.2 which is in buster.

To set this up, you need to add the following two lines to your
``~/.gnupg/gpg-agent.conf`` file, creating it if it doesn't exist::

  use-standard-socket
  enable-ssh-support

You can omit the first line on Debian buster as it is the default there.

You then need to logout and log back into your session for the changes to take
effect.

If your Xfce setup uses a systemd user session, as is the default on Debian
stretch, you should see something like the following::

  $ echo $SSH_AUTH_SOCK
  /run/user/1000/gnupg/S.gpg-agent.ssh
  $ gpgconf --list-dirs agent-ssh-socket
  /run/user/1000/gnupg/S.gpg-agent.ssh
  $ systemctl --user show-environment | grep SSH_AUTH_SOCK
  SSH_AUTH_SOCK=/run/user/1000/gnupg/S.gpg-agent.ssh

If so, you're all set!
