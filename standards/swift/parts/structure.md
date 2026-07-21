# Swift — Structure

> File/folder organization, naming, access control, and Xcode project registration.

## Do / Don't

- Name files `<TypeName>.swift` (one primary type per file); mirror `Sources/` under `Tests/`; group by feature/layer.
- `internal` by default; `public` only for library/framework API; `private` for implementation details.
- Prefix capability protocols `-able`/`-ible`; noun for "is-a" protocols. `UpperCamelCase` types, `lowerCamelCase` else.
- Don't mix unrelated types in one file, use `public` on internal app code, or make `Internal/` folders for access control.
- Don't hand-edit `project.pbxproj` with sed/text replace — one bad UUID corrupts the whole file.

## Xcode Project Registration

A file only "exists" to Xcode once **Reference** (`PBXFileReference`/`PBXGroup`), **Target** (`PBXBuildFile` in a build phase), and **Scheme** (target wired into a `.xcscheme`) are all true — not just written to disk.

1. **Sync check (Xcode 16+):** `grep -B2 'path = "?<Folder>"?;' *.xcodeproj/project.pbxproj | grep isa` — if `PBXFileSystemSynchronizedRootGroup`, Reference+Target are automatic, skip to step 3.
2. **Classic group — register via the `xcodeproj` gem:**
   ```bash
   gem install xcodeproj --no-document
   ruby -rxcodeproj -e '
     p = Xcodeproj::Project.open("<Proj>.xcodeproj")
     t = p.targets.find { |x| x.name == "<Target>" }
     g = p.main_group.find_subpath("<Path>", true)
     t.add_file_references([g.new_reference("<File>.swift")])
     p.save'
   ```
   Verify: `grep -c "<File>.swift" *.xcodeproj/project.pbxproj` (expect 2+).
3. **Scheme (new targets only):** confirm a `BuildableReference` exists in `xcshareddata/xcschemes/*.xcscheme`; watch for `<SelectedTests>` allow-lists silently skipping new test classes.

Gem unavailable → don't hand-edit the pbxproj; ask the user to add the file via Xcode instead.
New test *targets* need more wiring — see `testing.md` → Test Target Setup.

## Notes

- Default hierarchy: `Sources/App/{Models,Views,Services}` mirrored under `Tests/AppTests/`.
