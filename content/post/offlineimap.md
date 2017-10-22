---
title: Sync Evolution mail with offlineimap
date: 2015-02-08
aliases:
    - /linux/2015/02/08/offlineimap/
---

This is 2015 version of François Michonneau's post
[Evolution 3 and offlineimap][2011].
It contains a few improvements to offlineimap config as well
as some extra usefulness.

## Goal

Use [offlineimap][offlineimap] to sync [Evolution mail client][evolution]
with Gmail account through IMAP protocol.

## Specifics of offlineimap and Evolution maildir handling

In case when offlineimap uses dot as a separator for labels (`sep = .`),
Gmail labels are synced in a following way:

    INBOX
    Label1
    Label2
    Label2.Sublabel1
    Label2.Sublabel2.Subsublabel

Evolution mail client handles maildir in a different way. It replaces dots in
label names with '_2E', adds dot to all labels, and uses root directory for Inbox:

    cur  \
    new  |- INBOX
    tmp  /
    .Label1
    .Label2
    .Label2.Sublabel1
    .Label2.Sublabel2.Subsublabel

Offlineimap nametrans rules are required to convert labels between these two formats.

for IMAP repository:

    nametrans = lambda folder: re.sub('^.INBOX$', '',
                               re.sub('^', '.',
                               re.sub('\.', '_2E',
                               re.sub('^\[Gmail\].Drafts$', 'Drafts',
                               re.sub('^\[Gmail\].Sent Mail$', 'Sent', folder)))))

for Local repository (to translate labels back to format used by Gmail):

    nametrans = lambda folder: re.sub('^Sent$',   '[Gmail].Sent Mail',
                               re.sub('^Drafts$', '[Gmail].Drafts',
                               re.sub('_2E', '.',
                               re.sub('^.', '',
                               re.sub('^$', '.INBOX', folder)))))

Evolution uses Outbox label for not-yet-sent email: it should be excluded from sync.
When Outbox label (directory with `.Outbox` name) is not present in the root directory,
Evolution could not recognize maildir and proposes to convert it from mbox format (??).
Also, the ways how Evolution and Gmail handle Trash label are slightly different,
so I cannot find a way to sync it through offlineimap (there is no point to do it anyway).

There is important difference between Maildir account manually added to Evolution
and built-in "On This Computer" account (the latter is just a maildir
stored in `~/.local/share/evolution/mail/local/` directory). When some new label
arrives in custom-added maildir account, Evolution immediately renames it to its builtin
format. So, when one creates label `Label3/Sublabel3` in Gmail, offlineimap converts its
name to `.Label3.Sublabel3`, AND then Evolution thinks it is not yet renamed and converts
it to `._2ELabel3_2ESublabel3`. Of course it is not what is needed, so I use built-in
"On This Computer" account to store mail synced with offlineimap.


## Configuration steps

* Start Evolution once to create necessary tree layout inside
   `~/.local/share/evolution/mail/local/` directory.

* Replace directory with symlink to more appropriate location:

        mv $HOME/.local/share/evolution/mail/local/ $HOME/mail
        ln -s $HOME/.local/share/evolution/mail/local $HOME/mail

3. Create offlineimap config in `~/.config/offlineimap/config`:

        [general]
        accounts = Gmail
        pythonfile = ~/.config/offlineimap/offlineimap.py

        [Account Gmail]
        localrepository = Local
        remoterepository = IMAP
        status_backend = sqlite
        synclabels = yes
        labelsheader = X-Label

        [Repository Local]
        type = GmailMaildir
        localfolders = ~/mail
        sep = .
        nametrans = lambda folder: re.sub('^Sent$',   '[Gmail].Sent Mail',
                                re.sub('^Drafts$', '[Gmail].Drafts',
                                re.sub('_2E', '.',
                                re.sub('^.', '',
                                re.sub('^$', '.INBOX', folder)))))

        folderfilter = lambda foldername: foldername not in ['.Outbox']

        [Repository IMAP]
        type = Gmail
        sslcacertfile = /etc/ssl/certs/ca-certificates.crt
        remoteusereval = keyring.get_password("Gmail", "username")
        remotepasseval = keyring.get_password("Gmail", "password")
        usecompression = yes
        nametrans = lambda folder: re.sub('^.INBOX$', '',
                                re.sub('^', '.',
                                re.sub('\.', '_2E',
                                re.sub('^\[Gmail\].Drafts$', 'Drafts',
                                re.sub('^\[Gmail\].Sent Mail$', 'Sent', folder)))))
        folderfilter = lambda folder: folder not in [
            '[Gmail]/Starred', '[Gmail]/Important','[Gmail]/Chats',
            '[Gmail]/All Mail', '[Gmail]/Spam', '[Gmail]/Trash']

Full offlineimap config with comments is available at my [dotfiles][github] repo.

4. Use python-keyring to store Gmail login/password:

        echo 'import keyring' >> ~/.config/offlineimap/of
        python2
        >>> import keyring
        >>> keyring.set_password("Gmail", "username", "yourusername@gmail.com")
        >>> keyring.set_password("Gmail", "password", "yourpassword")

5. Check everything works:

        offlineimap

[2011]: http://francoismichonneau.net/2011/06/evolution-3-and-offlineimap/
[offlineimap]: http://offlineimap.org/
[evolution]: https://wiki.gnome.org/Apps/Evolution
[github]: https://github.com/medvid/dotfiles/blob/master/config/offlineimap/config
