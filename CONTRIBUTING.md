# Contributing a site package

7x and the Exponential Foundation strongly encourage every developer, designer and partner who has built a site
package to submit it here by pull request. This repository is the package server
[exponential.packages.exponential.earth](https://exponential.packages.exponential.earth/): every push to `master`
publishes it through GitHub Pages, and every Exponential installation reads its `index.xml`. An accepted package is
offered by the setup wizard, the kickstarter, `exp:install --package=<name>` and ezupdate on every site, with no
extra settings on anyone's side.

You keep your own source repository and your own copyright; the pull request adds the built package and its index
entry. The full guide, with the reasons behind each rule, is the help page
[Contribute your site package](https://github.com/se7enxweb/exponential/blob/main/doc/help/site-packages/contribute-your-site-package.md)
(`doc/help/site-packages/contribute-your-site-package.md` in every Exponential installation).

## 1. Fork and branch

```bash
gh repo fork se7enxweb/exponential.packages.exponential.earth --clone
cd exponential.packages.exponential.earth
git checkout -b add-<your_package>
```

The repository holds every built archive of every release and is large; a partial clone is enough:
`git clone --filter=blob:none <your fork's URL>`.

Keep the **source** of your package (the unpacked folders) in a repository of your own, the way the 7x packages
live in [se7enxweb/sevenx-ezpackage](https://github.com/se7enxweb/sevenx-ezpackage) and the CJW packages in
[se7enxweb/cjw_multisite-ezpackage](https://github.com/se7enxweb/cjw_multisite-ezpackage). Reviewers will read it.

## 2. Folder layout

One folder per Exponential release. Your files go into the folder of each release you support, next to the
existing packages:

```text
exponential/6.0/6.0.15/
├── index.xml                          add one <package> line per package
├── <your_package>.ezpkg               the site package (type site)
├── <your_package>_democontent.ezpkg   the packages it requires, if any
└── thumbnails/
    └── <your_package>.png             the picture the setup wizard shows
```

- The file name is `<name>.ezpkg`, with `<name>` exactly as in the package's `package.xml`: lower case letters,
  digits and `_`.
- A name of your own. An import refuses a package whose name already exists.
- Every package your site package requires is on the server too, in the same folder.
- Built archives only: a gzip compressed tar of the package folder's contents, `package.xml` at the top, without
  `.git`, editor files, backup copies (`*.bak`) and `node_modules`. The `build.sh` of
  [sevenx-ezpackage](https://github.com/se7enxweb/sevenx-ezpackage/blob/master/packages/build.sh) shows how.

## 3. The index entry

Add your packages to the release's `index.xml`, in the form of the existing entries. A required package gets a short
entry in the dependency list at the top; the site package gets a full one:

```xml
<package name="<your_package>_democontent" type="contentobject"
         url="https://exponential.packages.exponential.earth/exponential/6.0/6.0.15/<your_package>_democontent.ezpkg"/>

<package name="<your_package>" version="1.0-1" type="site"
         summary="<one line, shown as the package's title>"
         description="<two or three sentences: what the site is and what it contains>"
         url="https://exponential.packages.exponential.earth/exponential/6.0/6.0.15/<your_package>.ezpkg"
         thumbnail_url="https://exponential.packages.exponential.earth/exponential/6.0/6.0.15/thumbnails/<your_package>.png"/>
```

The URLs must be absolute and correct; the wizard uses every attribute as it is and lists only `type="site"`
entries as choices. Check the file with `xmllint --noout index.xml`.

## 4. Versions

- A package's version is `<version><number>` and `<release>` in its `package.xml`. The `version` attribute of the
  index entry is that version written the way the kernel writes it, `<number>-<release>`: `<number>1.0</number>`
  and `<release>1</release>` are listed as `version="1.0-1"`. The 7x site packages have `<number>6.0</number>` and
  `<release>15-stable</release>`, so they are listed as `version="6.0-15-stable"`.
- Keep the two the same. The setup wizard compares the index version with a copy already imported on the site: a
  higher index version makes it download the package again every time, a lower one hides your new release.
- A changed package gets a new version: raise `<release>` (or `<number>`) when you replace its file, and say what
  changed in `<changelog>` and in the pull request.
- A tag pushed in your own repository is never moved; publish the next version instead.

## 5. Requirements checklist

- [ ] `package.xml` is well formed, has `<package>` as its root and no document type.
- [ ] `<name>` matches the file name.
- [ ] `<summary>` and `<description>` say what the site is, in plain words.
- [ ] `<vendor>` names you or your organization.
- [ ] `<type value="site" />` for the site package, the right type for each package it requires.
- [ ] `<version>` agrees with the index entry (section 4).
- [ ] `<ezpublish><version>` names the Exponential release you tested with (the element keeps its historical name).
- [ ] `<licence>` holds a license identifier (`GPL-2.0-or-later` is the default), and `documents/LICENCE` holds the
      license text and names **this** package on its first line.
- [ ] `<changelog>` has an entry for this release; `<maintainers>` lists a maintainer.
- [ ] `<dependencies><requires>` lists every package it needs, and each is in the pull request or already here.
- [ ] `files/default/thumbnail/thumbnail.png` exists, and `thumbnails/<your_package>.png` is in the pull request.
- [ ] Every `md5sum` attribute in `package.xml` matches its file (`md5sum files/default/*/*`).
- [ ] No secrets: no passwords, keys, tokens or real e-mail addresses in install scripts, settings or content.
- [ ] The package passes **Validate** on its page in **Setup > Packages** of an installation.

## 6. License

- **Code** in the package (install scripts, templates, settings, extensions): a license compatible with the GNU GPL,
  because Exponential is GPL 2.0 or later.
- **Documentation** shipped with it: the GNU Free Documentation License 1.3, as for Exponential's own documentation.
- **Images and demo content**: only material you have the right to share, with its license stated.

## 7. Test the install in a sandbox

Never test a site package on a site with real content: installing it builds a new site.

1. Make a throwaway installation, for example a fresh clone with its own SQLite database.
2. Point it at your copy of the release folder: serve the folder (for example `php -S 127.0.0.1:8090 -t <folder>`),
   point the index URLs at it, and set `RemotePackagesIndexURL` in the sandbox's
   `settings/override/package.ini.append.php`:

   ```ini
   [RepositorySettings]
   RemotePackagesIndexURL=http://127.0.0.1:8090/6.0.15
   ```

   (or import the archives into the sandbox's local repository).
3. Install with the setup wizard and once more with `./console exp:install --db=sqlite --package=<your_package>`,
   so both front ends are proven. The install must end with no errors.
4. Check the site, the administration, the roles, a search, an edit and a publish, and read `var/log/error.log`.
5. Remove the sandbox.

## 8. Open the pull request

```bash
git add exponential/6.0/6.0.15/<your_package>*.ezpkg exponential/6.0/6.0.15/thumbnails/<your_package>.png exponential/6.0/6.0.15/index.xml
git commit -m "Added: <your_package> site package for 6.0.15"
git push origin add-<your_package>
gh pr create -R se7enxweb/exponential.packages.exponential.earth --base master
```

Commit subjects start with `Added:`, `Updated:`, `Removed:` or `Renamed:`. In the pull request, write:

| Section | Content |
|---|---|
| What it is | The kind of site, its siteaccesses and languages, what an editor gets |
| What it contains | The packages and their types, and the extensions and designs it needs |
| Source | The link to the repository with the unpacked source |
| Tested | The Exponential version, the database(s) and the front ends you installed with |
| License | The license of the code, of the documentation and of the images |
| Maintainer | Who answers issues about the package, and where |

## 9. Review

The maintainers of this repository review the pull request: they read the source repository, check the archive and
the index entry against the checklist, install the package in a sandbox of their own, and may ask for changes. When
it is merged, the push to `master` republishes the server, and every installation that reads that release's index
offers your package.

Questions before you open a pull request are welcome in the issues of this repository.
