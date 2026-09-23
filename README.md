About
----- 

**drawio-desktop** is a diagramming desktop app based on [Electron](https://electronjs.org/) that wraps the [core draw.io editor](https://github.com/jgraph/drawio).

Private ArchiMate 4 fork
-------------------------

This is a private fork of draw.io Desktop for local ArchiMate 4 modelling. It
is not an official draw.io distribution and is not intended for upstream
contribution or support.

The fork currently adds the following ArchiMate 4 workflow improvements:

- A relationship chooser when a Quick Connect arrow is dragged between two
  ArchiMate 4 elements. It includes the standard relationship types, including
  the three Access notations: unspecified, read or write, and read/write.
- Junction-aware Quick Connect. The first relationship connected to an And or
  Or junction selects the relationship type. Later connections inherit that
  type, including connections made through the Shape Picker. Junction-to-
  junction connections are deliberately blocked.
- An ArchiMate 4 Shape Picker containing the semantic ArchiMate elements,
  without a duplicate of the source element or the Grouping element. Generic
  Common elements are represented once, while the domain-specific variants
  remain available. And and Or junctions are included when starting from a
  regular ArchiMate element.
- Larger default And and Or junctions, both in the sidebar and when created
  from the Shape Picker.

Relationship and target-element validity is not yet checked against the
ArchiMate relationship matrix. That is a planned enhancement; users must
currently choose a semantically valid relationship themselves.

The implementation is primarily in
`drawio/src/main/webapp/js/diagramly/ArchiMateQuickConnect.js`, with related
ArchiMate shape definitions in
`drawio/src/main/webapp/js/diagramly/sidebar/Sidebar-ArchiMate4.js`.

Run the private fork in development mode on Windows:

```powershell
cd C:\Repositories\drawio-desktop
$env:DRAWIO_ENV = "dev"
npm start
```

Close any official draw.io Desktop instance before starting it, then open
diagrams from this development instance. This avoids Windows forwarding the
launch to the installed application instead of the fork. The fork window title
ends with ` (ArchiMate4-fork)`.

### Run and distribute the private Windows fork

The fork has its own Windows app id, product name and user-data directory. It
deliberately registers no file associations, so the
official app remains the default for `.drawio`, `.vsdx`, `.mmd` and `.mermaid`
files. The fork can therefore run alongside an official
draw.io installation.

Build the private Windows fork:

```powershell
npm run sync -- disableUpdate
npm run build:archimate4-win
```

The tested portable application is the complete directory
`dist/archimate4-fork/win-unpacked/`. Start
`draw.io ArchiMate4-fork.exe` from that directory, and keep all its adjacent
files together. It can be copied or zipped and shared without installation.

The build configuration also targets an optional per-user NSIS installer. Only
distribute one when the build has actually produced
`draw.io ArchiMate4-fork-<version>-windows-installer.exe` in
`dist/archimate4-fork/`. An unsigned installer may require *More info* and
*Run anyway* in SmartScreen. It has a separate product identity and does not
replace the official draw.io installation.

Automatic updates are disabled for this private fork. For a shared release,
publish the source and artifacts from a repository you control, create a tag
for each version, and attach either the portable ZIP, the verified installer,
or both to that repository's release. Do not use the upstream draw.io release
page or its publishing scripts for fork releases.

**Can I use this app for free?** Yes, under the apache 2.0 license. If you don't change the code and accept it is provided "as-is", you can use it for any purpose.

Windows installation
--------------------

Three flavours of Windows download are published on the [releases page](https://github.com/jgraph/drawio-desktop/releases):

- `draw.io-<version>-windows-installer.exe` — NSIS installer. Installs **per-machine** into `Program Files` and **requires administrator privileges**.
- `draw.io-<version>.msi` — MSI installer. Installs **per-user** into the user's profile and **does not require administrator privileges**. Use this one if you don't have admin rights on your machine.
- `draw.io-<version>-windows-no-installer.exe` — portable build that runs without any installation (and therefore without admin rights). File-type associations are not registered.

The Microsoft Store (APPX) build is also installable per-user without admin rights via the Store.

### Windows on Arm

draw.io Desktop is built natively for Windows on Arm (ARM64) and is supported on Windows 11 ARM64 devices. Two native ARM64 downloads are published with every release:

- `draw.io-arm64-<version>-windows-arm64-installer.exe` — NSIS installer, per-machine, requires administrator privileges.
- `draw.io-arm64-<version>-windows-arm64-no-installer.exe` — portable build, no installation or admin rights needed.

The MSI and Microsoft Store builds are x64 only and run under emulation on ARM64 devices. ARM64 builds up to and including 31.4.4 were shipped with auto-update disabled; install a newer release manually once, after which the ARM64 build updates itself like x64.

Linux installation
------------------

If you manage AppImages with [AppImageLauncher](https://github.com/TheAssassin/AppImageLauncher), you need a 3.0 release of it (currently labelled beta). Since 31.4.2 the AppImage uses the static AppImage runtime so that it no longer depends on the end-of-life `libfuse2`, and AppImageLauncher 2.2.0, the last stable release, cannot load a static runtime. The app then fails to start with:

```
fuse: memory allocation failed
squashfuse 0.5.2 (c) 2012 Dave Vasilevsky
...
Can't open squashfs image: Bad address
```

Install a current AppImageLauncher from its [releases page](https://github.com/TheAssassin/AppImageLauncher/releases), which provides .deb packages, or uninstall AppImageLauncher altogether. It is not needed to run the AppImage. See [#2538](https://github.com/jgraph/drawio-desktop/issues/2538) for the detail.

Security
--------

draw.io Desktop is designed to be completely isolated from the Internet, apart from the update process. This checks github.com at startup for a newer version and downloads it from an AWS S3 bucket owned by Github. To disable the update check entirely (e.g. for centrally-managed installs), set the `DRAWIO_DISABLE_UPDATE=true` environment variable or pass `--disable-update` on launch. All JavaScript files are self-contained, the Content Security Policy forbids running remotely loaded JavaScript.

No diagram data is ever sent externally, nor do we send any analytics about app usage externally. The Content Security Policy on the web part of the interface forbids remotely-loaded JavaScript and restricts the application's own network connections to itself, so the app cannot transmit your diagrams or otherwise phone home. Note that a diagram can reference external media - for example an image, background or font loaded from a URL embedded in the diagram - and these are fetched when the diagram is opened so that it renders correctly. Opening a diagram from an untrusted source may therefore trigger a request to the referenced URL, which can reveal metadata such as your IP address to that server; no diagram content is transmitted.

Security and isolating the app are the primarily objectives of draw.io desktop. If you ask for anything that involves external connections enabled in the app by default, the answer will be no.

Support
-------

Support is provided on a reasonable business constraints basis, but without anything contractually binding. All support is provided via this repo. There is no private ticketing support for non-paying users.

Purchasing draw.io for Confluence or Jira does not entitle you to commercial support for draw.io desktop.

Developing
----------

**draw.io** is a git submodule of **drawio-desktop**. To get both you need to clone recursively:

`git clone --recursive https://github.com/jgraph/drawio-desktop.git`

To run this:
1. `npm install` (in the root directory of this repo)
2. [internal use only] export DRAWIO_ENV=dev if you want to develop/debug in dev mode.
3. `npm start` _in the root directory of this repo_ runs the app. For debugging, use `npm start --enable-logging`.

Note: If a symlink is used to refer to drawio repo (instead of the submodule), then symlink the `node_modules` directory inside `drawio/src/main/webapp` also.

To fork the project, make your own changes and build an (unsigned) app for personal use, see [doc/BUILDING_FOR_PERSONAL_USE.md](doc/BUILDING_FOR_PERSONAL_USE.md).

To release:
1. Update the draw.io sub-module and push the change. Add version tag before pushing to origin.
2. Wait for the builds to complete (https://travis-ci.org/jgraph/drawio-desktop and https://ci.appveyor.com/project/davidjgraph/drawio-desktop)
3. Go to https://github.com/jgraph/drawio-desktop/releases, edit the preview release.
4. Download the windows exe and windows portable, sign them using `signtool sign /a /tr http://rfc3161timestamp.globalsign.com/advanced /td SHA256 c:/path/to/your/file.exe`
5. Re-upload signed file as `draw.io-windows-installer-x.y.z.exe` and `draw.io-windows-no-installer-x.y.z.exe`
6. Add release notes
7. Publish release

Local Storage and Session Storage is stored in the AppData folder:

- macOS: `~/Library/Application Support/draw.io`
- Windows: `C:\Users\<USER-NAME>\AppData\Roaming\draw.io\`

Not open-contribution
---------------------

draw.io is closed to contributions (unless a maintainer permits it, which is extremely rare).

The level of complexity of this project means that even simple changes 
can break a _lot_ of other moving parts. The amount of testing required 
is far more than it first seems. If we were to receive a PR, we'd have 
to basically throw it away and write it how we want it to be implemented.

We are grateful for community involvement, bug reports, & feature requests. We do
not wish to come off as anything but welcoming, however, we've
made the decision to keep this project closed to contributions for 
the long term viability of the project.
