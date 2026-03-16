# Known Issues (Current)

Updated: 2026-02-23

## 1. Editing Features
- Interpolation curve editing is implemented, but multiple key simultaneous editing and copy/paste are not supported.
- Actual value/interpolation snapshot saving when adding camera keys is implemented, but range editing UI is not implemented.
- Property (display/IK) track editing is not implemented.
- VMD export is not implemented.
- Project JSON is implemented, but specification documentation for external tool integration is insufficient.

## 2. Timeline Representation
- Camera rows share the same frame column, and independent key editing by channel is not supported.
- Key editing is centered on add/delete/1f move, and advanced range editing is not implemented.

## 3. Physics
- Operational specifications for rigid body modes 0/1/2 are undocumented.
- Design for `disableBidirectionalTransformation` equivalent switching is incomplete.
- Rigid body/constraint debug visualization is not implemented.

## 4. Technical Notes
- There are environments with editing risks due to character encoding in `src/mmd-manager.ts`.
- The repository shows CRLF conversion warnings, so pay attention to line ending changes when checking diffs.

## 5. Priority Candidates for Support
- 1. Property track editing
- 2. VMD export
- 3. Range operations for key editing (duplicate/paste/scale)
- 4. Automated rotation interpolation MMD actual machine comparison testing
