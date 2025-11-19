# CLAUDE.md - AI Assistant Guide for gpmf-extract

## Project Overview

**gpmf-extract** is a JavaScript/Node.js library that extracts GoPro Metadata Format (GPMF) telemetry data from GoPro video files (Hero5 and later). The library finds and extracts the metadata track from MP4/MOV video files, returning raw binary data and timing information.

**Key Characteristics:**
- Dual environment support (Node.js and browser)
- Minimal dependencies (only mp4box for MP4 parsing)
- TypeScript definitions via JSDoc and index.d.ts
- ~318 lines of source code (compact, focused implementation)

## Codebase Structure

```
eos-gpmf-extract/
├── index.js              # Main library entry point (194 lines)
├── index.d.ts            # TypeScript type definitions
├── code/
│   ├── readBlock.js      # File chunk reader (browser/Node.js)
│   └── inline-worker.js  # Web Worker factory for browser
├── tests/
│   ├── node.test.js      # Node.js environment tests
│   ├── browser.test.js   # Browser tests with Puppeteer
│   └── browser/
│       └── index.html    # Browser test harness
├── samples/
│   ├── example.js        # Usage example
│   ├── karma.mp4         # Test video file (4.1 MB)
│   └── karma.raw         # Expected extraction output
├── .circleci/
│   └── config.yml        # CI/CD configuration
├── jest.config.js        # Jest test configuration
└── package.json          # Project manifest
```

## Development Setup

### Prerequisites
- Node.js 20.x or later
- npm

### Installation
```bash
npm install
```

### Build for Browser Testing
```bash
npm run build
```
This bundles `index.js` to `tests/browser/dist.js` using esbuild.

### Run Tests
```bash
npm test
```
This builds the browser bundle and runs both Node.js and browser tests via Jest/Puppeteer.

## Architecture & Key Patterns

### Dual Environment Support

The library supports both Node.js and browser environments with different input types:

**Node.js:**
- Input: `Buffer` or callback function `(file: ISOFile) => void`
- Uses native Node.js streams
- No Web Worker support

**Browser:**
- Input: `File` or `Blob` objects
- Uses Streams API or blob streams
- Optional Web Worker for non-blocking extraction (`useWorker: true`)
- Supports cancellation tokens

### Core Flow (index.js)

1. Initialize MP4Box parser
2. Read file in chunks using `readBlock.js`
3. Parse moov box to find GPMF track (handler type "meta", codec type "gpmd")
4. Extract timing information from track samples
5. Concatenate raw metadata payloads
6. Return structured result with rawData and timing

### Module Responsibilities

| File | Purpose |
|------|---------|
| `index.js` | Main extraction logic, MP4Box integration |
| `code/readBlock.js` | Cross-platform file chunk reading |
| `code/inline-worker.js` | Browser Web Worker setup |

## Code Conventions

### Style Guidelines (No Automated Linting)

- **Quotes:** Double quotes (`"`)
- **Indentation:** 2 spaces
- **Variables:** Mix of `var`, `const`, `let` (prefer `const`/`let` for new code)
- **Functions:** Traditional function declarations (limited arrow functions)
- **Naming:** camelCase for variables and functions
- **Module System:** CommonJS (`require`/`module.exports`)

### JSDoc Type Annotations

Use JSDoc comments for type hints in JavaScript:
```javascript
/**
 * @param {Buffer|Function} file
 * @param {Object} options
 * @returns {Promise<Object>}
 */
function GPMFExtract(file, options) { ... }
```

### TypeScript Definitions

When adding new options or changing the API:
1. Update JSDoc comments in `index.js`
2. Update type definitions in `index.d.ts`
3. Maintain both Node.js and browser overloads

## Testing

### Test Structure

- **Node tests** (`tests/node.test.js`): Buffer input, stream reading, framerate/duration extraction
- **Browser tests** (`tests/browser.test.js`): File/Blob input, worker/main thread execution

### Running Specific Tests
```bash
# All tests
npm test

# Just build (for browser manual testing)
npm run build
```

### Test Data
- Source video: `samples/karma.mp4` (GoPro footage)
- Expected output: `samples/karma.raw` (reference metadata)

### Adding Tests

1. Place test files in `tests/` with `.test.js` extension
2. Update `jest.config.js` if adding new test projects
3. For browser tests, ensure they work with Puppeteer

## Common Development Tasks

### Adding a New Option

1. Add parameter handling in `index.js`
2. Update JSDoc comments
3. Update `index.d.ts` type definitions (both overloads if needed)
4. Add tests for the new option
5. Update `readme.md` documentation

### Modifying Extraction Logic

1. Work in `index.js` main extraction function
2. Test with both Node.js and browser environments
3. Verify output matches `samples/karma.raw` reference

### Browser-Specific Changes

1. Modify `code/readBlock.js` for file reading changes
2. Modify `code/inline-worker.js` for Worker behavior
3. Test with `useWorker: true` and `useWorker: false`

### Updating Dependencies

1. Update `package.json`
2. Run `npm install`
3. Run full test suite
4. Verify browser bundle still works

## Git Workflow

### Branch Strategy
- **Main branch:** Production releases
- **dev branch:** Active development
- Feature branches merge to dev first

### Contributing
1. Create feature branch from dev
2. Implement changes with tests
3. Run `npm test` to verify
4. Submit PR to dev branch

### Commit Messages
Use clear, descriptive commit messages:
- `Add support for X feature`
- `Fix Y extraction issue`
- `Update dependencies to version Z`

## CI/CD

CircleCI runs on every commit:
1. Uses `cimg/node:20.0.0-browsers` Docker image
2. Installs dependencies
3. Runs full test suite (Node.js + Browser)

## API Reference

### Main Function

```javascript
const gpmfExtract = require('gpmf-extract');

// Node.js
const result = await gpmfExtract(buffer, options);

// Browser
const result = await gpmfExtract(file, options);
```

### Options

| Option | Type | Environment | Description |
|--------|------|-------------|-------------|
| `browserMode` | boolean | Both | Force browser mode detection |
| `useWorker` | boolean | Browser | Use Web Worker for extraction |
| `progress` | function | Both | Progress callback (percent) |
| `cancellationToken` | object | Browser | Cancel extraction with `.cancel()` |

### Result Structure

```javascript
{
  rawData: Uint8Array | Buffer,  // Extracted GPMF data
  timing: {
    samples: Array<{cts, duration}>,
    framesDuration: number,
    videoDuration: number,
    frameDuration: number
  }
}
```

## Troubleshooting

### Common Issues

1. **Browser tests fail:** Ensure Chromium is available (Puppeteer dependency)
2. **Build errors:** Check Node.js version (20.x required)
3. **Type errors:** Verify `index.d.ts` matches actual implementation

### Debug Tips

- Check browser console in `tests/browser/index.html`
- Add console.log in `index.js` for extraction flow
- Use Puppeteer's non-headless mode for visual debugging

## Related Projects

- [gopro-telemetry](https://github.com/JuanIrache/gopro-telemetry) - Parses extracted GPMF data
- [mp4box.js](https://github.com/nicholasStb/nicholasStb.github.io) - MP4 parsing library

## Important Notes for AI Assistants

1. **Keep changes minimal** - This is a focused library; avoid feature creep
2. **Test both environments** - Always verify Node.js and browser work
3. **Update types** - Keep `index.d.ts` in sync with any API changes
4. **Reference data is truth** - Tests compare against `samples/karma.raw`
5. **No linting tools** - Manually follow existing code style
6. **Dual exports** - Maintain both `module.exports` and `exports` patterns
