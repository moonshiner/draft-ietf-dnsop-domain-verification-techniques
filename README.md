# Domain Control Validation using DNS

This is the working area for the DNSOP Internet-Draft, "Domain Control Validation using DNS" (previously called "Domain Verification Techniques using DNS").

[Datatracker.](https://datatracker.ietf.org/doc/draft-ietf-dnsop-domain-verification-techniques/)

* [Editor's Copy](https://ietf-wg-dnsop.github.io/draft-ietf-dnsop-domain-verification-techniques/#go.draft-ietf-dnsop-domain-verification-techniques.html)

The repo uses Martin Thomson's [I-D template](https://github.com/martinthomson/i-d-template).

## Building the Draft

Formatted text and HTML versions of the draft can be built using `make`.

```sh
$ make
```

This requires that you have the necessary software installed.  See
[the instructions](https://github.com/martinthomson/i-d-template/blob/master/doc/SETUP.md). For this I-D, Markdown is the canonical source, so `kramdown-rfc2629` is what I use. This must be installed in addition to the right version of `make` and `xml2rfc` (yay, software) - please check the [SETUP doc](https://github.com/martinthomson/i-d-template/blob/master/doc/SETUP.md) for details. 

Also note that `main` is the default and working branch.

## Quick commands

Once you have everything set up correctly, the usual flow is:

```sh
$ make
```

This generates the `.txt` and `.html` outputs. 

To automatically fix lints:
```sh
$ make fix-lint
```

Then do a `git commit`. Note that a git commit will typically fail if the lint check fails.

### Submitting

The current Datatracker version is `12`, so the next version will be `13`. Run `make next`
to confirm the next version number, and substitute it for `13` in the commands below.

There are two ways to cut a new version. Either way, build first (`make`) and make sure
you are happy with the output.

#### Automated submission (via a tag)

Push your commits, then push an *annotated* tag named for the new version. The
`publish.yml` GitHub Action builds the versioned draft and uploads it to the Datatracker
automatically; you will then receive an email asking you to confirm the submission.

```sh
git push origin main
git tag -a draft-ietf-dnsop-domain-verification-techniques-13 -m "version 13"
git push origin draft-ietf-dnsop-domain-verification-techniques-13
```

Use an annotated tag (`-a`) so that your email address is associated with the submission.
You can also run this manually from the repo's "Actions" tab ("Publish New Draft
Version"), where you can supply the submitter email directly.

#### Manual submission

Generate the versioned document locally and upload the XML through the web form:

```sh
make next # this generates the versioned doc under versioned/
# Upload the generated versioned/draft-ietf-dnsop-domain-verification-techniques-13.xml at
# https://datatracker.ietf.org/submit/
# Then push the tag to GitHub so the next version can be generated correctly:
git tag draft-ietf-dnsop-domain-verification-techniques-13
git push origin draft-ietf-dnsop-domain-verification-techniques-13
```

Both processes are described in the [I-D template submission docs](https://github.com/martinthomson/i-d-template/blob/main/doc/SUBMITTING.md). 

## Contributing Guidelines

See the
[guidelines for contributions](https://github.com/ietf-wg-dnsop/draft-ietf-dnsop-domain-verification-techniques/blob/main/CONTRIBUTING.md).
