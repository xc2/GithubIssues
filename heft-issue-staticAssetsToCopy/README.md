
## Summary

Files in `staticAssetsToCopy` are always copied to `lib` but not the one specifed in `tsconfig.json`.

## How to reproduce

```
# assign `outDir` to `dist` in `tsconfig.json`, and then
pnpm build
```

**Expected result:** `hello.js` and `data.json` in `dist` directory.
**Actual result:** `hello.js` in `dist`, and `data.json` in `lib`