# Undo / Redo Investigation Notes

Updated: 2026-03-10
Target:
- `src/ui-controller.ts`
- `src/mmd-manager.ts`
- `src/timeline.ts`

## 1. Purpose

- Investigate whether "undo/redo" close to current MMD can be introduced in stages.
- Before introducing libraries, organize realistic implementation units in this codebase.

## 2. Premise

- Currently, many edits directly call `MmdManager` from `UIController`, and there is no single entry point to capture history in bulk.
- Timeline keyframe arrays have many implementations that replace in immutable style, making them easy to use as foundation for history management.
- On the other hand, actual edit results are reflected to multiple layers: Babylon.js runtime state, source animation, and timeline display.

Related:
- [timeline-spec.md](./timeline-spec.md)
- [keyframe-storage-spec.md](./keyframe-storage-spec.md)
- [edit-state-machine.md](./edit-state-machine.md)

## 3. Core of Difficulty

- Even with React, undo/redo doesn't become simple unless state updates are aggregated to reducer/store.
- SQLite WASM is not suitable for save/restore per operation, and is too heavy as main system to bear immediate rollback.
- Difficulty of Undo / Redo is not UI technology, but "how to define 1 operation" and "where to stack history".

## 4. Approach Methods

### 4-1. Overall Snapshot Method

- Content: Save entire project state per operation and restore.
- Advantage: Implementation is easy to understand.
- Disadvantage: Becomes heavy as model count and keyframe count increase. Weak for continuous operations.
- Judgment: Possible for initial verification, but not for production.

### 4-2. Difference History Method

- Content: Stack only minimum differences before/after edit.
- Advantage: Suited for practical operation. Good compatibility with timeline editing.
- Disadvantage: Difference type design needed. Must write undo application processing per target operation.
- Judgment: Production candidate.

### 4-3. Command Pattern Method

- Content: Stack operation objects holding `do()` / `undo()`.
- Advantage: Clear what counts as 1 operation. Easy to introduce in stages.
- Disadvantage: Implementation amount increases as operation types increase.
- Judgment: Realistic to combine with difference history method.

## 5. Adoption Policy Proposal

- Add 1 `HistoryManager`.
- 1 history item is lightweight command holding `label` and `undo` / `redo`.
- Command internals hold minimum differences as needed.
- Combine continuous inputs into 1 item.
  - Example: During slider drag, don't stack history, only stack on commit.

Image:

```ts
type HistoryEntry = {
  label: string;
  undo: () => void;
  redo: () => void;
  mergeKey?: string;
};
```

## 6. Operations to Target First

Priority order:

- Keyframe add
- Keyframe delete
- Keyframe `±1f` move
- Key value numeric change

Reasons:

- Large user impact.
- Operation entry points relatively easy to track even in current implementation.
- Close to minimum line to feel "current MMD-like".

## 7. Operations to Exclude from Restoration

- Project load/save
- Model load/delete
- VMD / VPD / camera VMD load
- Playback/stop/seek
- Post effects in general
- Operations dependent on distribution or startup environment

Including these in history targets makes boundaries ambiguous, so exclude in initial stage.

## 8. Stage Introduction Proposal

### Phase 1

- Support only timeline key add/delete/move
- Enable `Ctrl + Z` / `Ctrl + Y`
- Stack labeled history per edit

### Phase 2

- History key value changes
- History interpolation changes
- Support multi-frame operations

### Phase 3

- Bone operations
- Morph operations
- Camera operations
- Accessory operations

## 9. Implementation Notes

- Even if only Babylon runtime state is restored, need to align source animation and timeline display as well.
- VMD load and VPD application update multiple tracks simultaneously, so need to make history units larger.
- If continuous inputs are history-ized as is, stack expands immediately.
- Playback editing, editing during physics ON, camera target editing have side effects, so separate confirmation needed.

## 10. About Using SQLite WASM Proposal

- Technically possible, but not main solution for this case.
- SQLite is suited for autosave, work logs, crash recovery.
- Natural for Undo / Redo main system to hold history stack on memory.

Conclusion:

- `Undo / Redo core = in-memory history`
- `SQLite = auxiliary save destination`

## 11. Conclusion

- No need to introduce large libraries or DB first.
- First introduce limited to timeline editing with `HistoryManager + lightweight command + minimum difference`.
- If aiming for usage feeling close to current MMD, quite effective at Phase 1 points.

## 12. Reserved Items

- Whether to make Redo shortcut `Ctrl + Y` or align to MMD actual behavior
- Criteria to summarize multiple changes within same frame as 1 item
- Organize relationship with history for VMD additional load and future multiple VMD merge
