# dal-test — does Digital Asset Links credential sharing work for an unpublished app?

A small, self-contained experiment on a domain I control, testing how Android's
`delegate_permission/common.get_login_creds` relation behaves.

**The question.** Android's [Credential Manager
prerequisites](https://developer.android.com/identity/credential-manager/prerequisites)
document `get_login_creds` as the relation that "enables sharing credentials between your
website and your Android app". A now-deprecated Smart Lock page additionally required that an
app be "released in the public channel for associations to be picked up". The current
documentation carries no equivalent statement, and the relation is evaluated server-side by
the credential provider rather than on-device, so it cannot be answered by reading AOSP.

So: **is a Play-published app still required, and is the signing certificate actually the
deciding factor?**

## Design

`/.well-known/assetlinks.json` on this origin lists **two** package names against **one**
signing certificate fingerprint:

| Package | Listed fingerprint | Actually signed with | Expected |
|---|---|---|---|
| `com.vfresearch.dalprobe` | key **A** | key **A** | credential offered |
| `com.vfresearch.dalprobe.control` | key **A** | key **B** | nothing offered |

Both packages are listed, so the only semantic difference between the two applications is
**whether the APK's signing certificate matches the fingerprint in the statement.** Listing
only one package would have confounded "certificate mismatch" with "package absent", which is
a different claim.

Neither application is published on Google Play. Both are installed locally on an emulator.

## Files

- `.well-known/assetlinks.json` — the two statements
- `.nojekyll` — **required.** Without it GitHub Pages runs Jekyll, which does not publish
  dot-directories, and `.well-known/` returns 404
- `index.html` — a throwaway sign-in form, so a password can be saved for this origin. It
  posts nowhere; the page is static and there is no server to receive anything
- `signed-in.html` — the navigation target, which is what triggers the browser's save prompt

**The repository must be named `<username>.github.io`.** Digital Asset Links statements are
only honoured at the **origin root** — `https://<username>.github.io/.well-known/assetlinks.json`.
A project repository would serve them under `/<reponame>/`, where nothing looks for them.

## Reading the result

A **positive** is self-authenticating: a credential saved for this website appearing inside an
application that did not save it can only have arrived through the affiliation.

A **negative is not diagnostic.** `androidx.credentials` has no error type meaning "the
Digital Asset Links association was rejected" on the password path, so an association failure,
an absent saved credential, a filtered account and a cold affiliation cache all surface as the
same `NoCredentialException`. A null result therefore does not distinguish "the relation does
not work for unpublished apps" from "I set the experiment up wrong".

## Scope

Everything here is mine: my domain, my signing keys, my package names, and an invented
credential of no value. Nothing in this repository tests, describes, or depends on anyone
else's site or software.
