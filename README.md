# Subtitle Studio License Violation Investigation

Documentation of license compliance issues discovered in Subtitle Studio (subtitlestudio.ai) by Articue Tech Limited.

**Disclaimer:** This repository is created for educational and transparency purposes. All information is based on publicly available data and technical analysis.

## Table of Contents

- [Executive Summary](#executive-summary)
- [Timeline](#timeline)
- [Technical Findings](#technical-findings)
  - [GPL-3.0 Violation](#gpl-30-violation)
  - [MIT License Violation](#mit-license-violation)
  - [Model Verification](#model-verification)
- [Investigation Methodology](#investigation-methodology)
- [Developer Response](#developer-response)
- [Legal Analysis](#legal-analysis)
- [Impact](#impact)
- [Resources](#resources)
- [Contributing](#contributing)
- [License](#license)

## Executive Summary

This investigation documents multiple open-source license violations in Subtitle Studio, a commercial macOS application marketed as a "privacy-first AI subtitle generator" priced at $69 USD.

**Key Findings:**

- GPL-3.0 violation through bundled `ffmpeg-static` package in closed-source commercial software
- MIT license violation through use of whisper.cpp/OpenAI Whisper model without attribution
- Misleading marketing presenting free open-source AI model as proprietary "advanced AI"
- Lack of accountability for customers who purchased violating versions

**Current Status:**

- GPL-3.0 issue acknowledged by developer and claimed fixed in v1.5.0
- MIT license violation remains unaddressed as of last verification
- No refund, compensation, or explanation provided to existing customers

## Timeline

| Date       | Event                                                                                 |
| ---------- | ------------------------------------------------------------------------------------- |
| 2026-01-05 | Initial discovery and public disclosure of GPL-3.0 violation                          |
| 2026-01-05 | Discovery of identical SHA256 hash between app model and free Whisper large-v2        |
| 2026-01-06 | Developer response: "v1.5.3 removed FFmpeg-static, everyone be careful with licenses" |
| 2026-01-07 | Documentation of final analysis                                                       |

## Technical Findings

### GPL-3.0 Violation

**Affected Versions:** v1.0.0 through v1.4.0

**Package:** ffmpeg-static v5.3.0 (GPL-3.0-or-later)

**Violation:** Distributed GPL-licensed software in closed-source commercial application without:

- Providing source code
- Licensing entire application under GPL-3.0
- Including GPL license text in accessible location

**Evidence:**

```json
// package.json (v1.4.0)
{
  "name": "subtitle-studio",
  "version": "1.4.0",
  "dependencies": {
    "ffmpeg-static": "^5.3.0"
  }
}

// node_modules/ffmpeg-static/package.json
{
  "name": "ffmpeg-static",
  "version": "5.3.0",
  "license": "GPL-3.0-or-later"
}
```

**Note on "Buried License":** While LICENSE files exist in `node_modules/ffmpeg-static/` within the compressed ASAR archive, this does not satisfy GPL requirements. End users cannot access these files without technical tools (asar extraction), making them effectively inaccessible as required by GPL's "included in all copies" provision.

### MIT License Violation

**Affected Versions:** All versions (v1.0.0 to current)

**Component:** Whisper large-v2 model via whisper.cpp (MIT License)

**Violation:** Uses OpenAI Whisper model without:

- Copyright notice for whisper.cpp authors or OpenAI
- MIT license text
- Attribution anywhere in application or website
- Disclosure of open-source usage

**Distribution Method:**

- v1.0.0-1.1.0: Bundled in application
- v1.2.0+: Require user downloads from `https://assets.subtitlestudio.ai/models/model.bin`

**Important:** Moving the download to their own hosting does not fix the violation. They are still distributing the MIT-licensed model without attribution, whether bundled or hosted separately.

### Model Verification

**SHA256 Comparison:**

### Subtitle Studio's model

```
File:   model.bin
Size:   1,080,732,091 bytes
SHA256: 3a214837221e4530dbc1fe8d734f302af393eb30bd0ed046042ebf4baf70f6f2
```

### Official Whisper large-v2-q5_0

```
File:   ggml-large-v2-q5_0.bin
Size:   1,080,732,091 bytes
SHA256: 3a214837221e4530dbc1fe8d734f302af393eb30bd0ed046042ebf4baf70f6f2
```

**Result**: Byte-for-byte identical

### Marketing vs Reality

Website claims: "Powered by advanced AI"

Reality: Unmodified open-source Whisper large-v2 model, freely available at https://huggingface.co/ggerganov/whisper.cpp

## Investigation Methodology

This investigation can be independently verified by anyone with basic technical knowledge.

### Prerequisites

```bash
# Install asar for Electron app extraction
npm install -g @electron/asar

# Download v1.4.0 (violating version)
# https://assets.subtitlestudio.ai/releases/Subtitle%20Studio-arm64-1.4.0.dmg
```

### Reproduction Steps

**1. Extract Application:**

```bash
cd /Applications/Subtitle\ Studio.app/Contents/Resources/
asar extract app.asar ~/extracted-app
cd ~/extracted-app
```

**2. Verify GPL Violation (v1.4.0):**

```bash
# Check dependencies
cat package.json | grep ffmpeg-static
# Output: "ffmpeg-static": "^5.3.0"

# Verify license
cat node_modules/ffmpeg-static/package.json | grep license
# Output: "license": "GPL-3.0-or-later"

# Check for accessible GPL disclosure
# (No UI element, no documentation, no website mention)
```

**3. Verify Model Identity:**

```bash
# Calculate hash of their model
shasum -a 256 model.bin
# 3a214837221e4530dbc1fe8d734f302af393eb30bd0ed046042ebf4baf70f6f2

# Download official model
curl -L https://huggingface.co/ggerganov/whisper.cpp/resolve/main/ggml-large-v2-q5_0.bin -o whisper-official.bin
shasum -a 256 whisper-official.bin
# 3a214837221e4530dbc1fe8d734f302af393eb30bd0ed046042ebf4baf70f6f2
```

**Result**: Identical

**4. Verify No Attribution:**

- Open Subtitle Studio application
- Check menu/settings for "Licenses" or "Acknowledgments" section (none exists)
- Visit https://subtitlestudio.ai and search for "whisper" or "open source" (no mentions)

## Developer Response

Response posted on [Threads](https://www.threads.com/@mtmcy_ig/post/DTI0OoakiAb) at 2026-01-06 00:01:41 HKT:

**Original (Cantonese):**

> "Update: 1.5.3 版本已經移除 FFmpeg-static package，感激大家嘅關心！大家 vibe code 個陣都要小心 d license 野！另外新版本嘅推出包含一個 Burn in 字幕功能，可以 Edit 字幕樣式。雖然暫時未有 export video 功能，但未來會整，順便比大家 preview 定多謝大家支持！"

**Translation:**

> "Update: Version 1.5.3 has removed the FFmpeg-static package, thanks for everyone's concern! Everyone should be careful with license stuff when vibe coding! Additionally, the new version includes a Burn-in subtitle feature where you can edit subtitle styles. Although there's no export video feature yet, we'll add it in the future. Thanks for your support!"

**Response Analysis:**

- Implicitly acknowledges GPL violation through package removal
- No explicit apology to affected customers
- Minimizes severity with casual language
- No mention of MIT license violation
- No offer of refund or compensation
- Immediately pivots to promoting new features

## Legal Analysis

### Why GPL-3.0 Was Violated

GPL-3.0 requires anyone distributing GPL-licensed software to:

1. Provide complete source code to recipients
2. License the entire combined work under GPL-3.0
3. Make license terms readily accessible to users

Subtitle Studio failed all three requirements by including GPL-licensed `ffmpeg-static` in a closed-source commercial product.

**"Buried License" Issue:** Having LICENSE files in `node_modules/` inside a compressed ASAR archive does not satisfy GPL requirements. These files are:

- Inaccessible without technical tools
- Not visible to end users
- Not in "human-readable" form as GPL requires

Industry standard requires licenses to be accessible via application UI (e.g., About → Licenses menu).

### Why MIT License Is Violated

MIT License has only one requirement:

> "The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software."

Subtitle Studio violated this by:

1. Using the Whisper model without any copyright notice
2. Not including MIT license text
3. Not attributing whisper.cpp or OpenAI anywhere
4. Marketing the model as proprietary technology

**Still Violating:** Even in v1.2.0+ where users download the model separately, the violation continues because:

- The application code using whisper.cpp still needs attribution
- Hosting the model on their own servers (`assets.subtitlestudio.ai`) means they're still distributing it
- No attribution exists on their download page or in the application

### What Compliance Looks Like

**GPL Compliance:**

- Provide source code via written offer or direct inclusion
- License entire application under GPL-3.0, OR
- Remove GPL-licensed components

**MIT Compliance:**

- Add "Acknowledgments" or "Open Source Licenses" section in app UI
- Include: "This app uses whisper.cpp (Copyright 2023-2024 The ggml authors) and OpenAI Whisper (Copyright OpenAI) under MIT License"
- Include full MIT license text
- Disclose on website that app uses open-source Whisper model

## Impact

### For Customers

- Paid $69 for repackaged free open-source software
- Purchased versions with license violations
- No notification, refund, or explanation provided
- Misled about "proprietary AI" technology

### For Open Source Authors

- whisper.cpp and ggml authors: No attribution for their work
- FFmpeg developers: GPL terms violated
- OpenAI: No attribution for Whisper model

### For Software Industry

- Demonstrates "move fast, fix when caught" approach to licensing
- Shows need for automated license compliance checking
- Highlights gap between permissive licenses and actual attribution practices

## Resources

**Archived Evidence:**

- Website (2026-01-06 GMT): https://web.archive.org/web/20260106175103/https://subtitlestudio.ai/
- Developer Response: https://www.threads.com/@mtmcy_ig/post/DTI0OoakiAb

**Official Sources:**

- Subtitle Studio: https://subtitlestudio.ai
- Developer Company: https://articue.io
- Download Link (v1.4.0): https://assets.subtitlestudio.ai/releases/Subtitle%20Studio-arm64-1.4.0.dmg

**Open Source Projects:**

- whisper.cpp: https://github.com/ggml-org/whisper.cpp (MIT)
- OpenAI Whisper: https://github.com/openai/whisper (MIT)
- ffmpeg-static: https://github.com/eugeneware/ffmpeg-static (GPL-3.0-or-later)
- FFmpeg: https://ffmpeg.org (GPL-2.0+/LGPL-2.1+)

**License Texts:**

- GPL-3.0: https://www.gnu.org/licenses/gpl-3.0.html
- MIT License: https://opensource.org/license/mit

**Tools:**

- asar: https://github.com/electron/asar
- Whisper Models: https://huggingface.co/ggerganov/whisper.cpp

## Contributing

Found additional information or errors? Please open an issue or submit a pull request.

**Guidelines:**

- Provide verifiable evidence (links, screenshots, technical data)
- Maintain factual, respectful tone
- No personal attacks or harassment

## License

This documentation is released under CC0 1.0 Universal (Public Domain). See [LICENSE](LICENSE) file for details.

---

**Last Updated:** 2026-01-07  
**Investigator:** [@nelsonlaidev](https://github.com/nelsonlaidev)  
**Contact:** me@nelsonlai.dev

**Disclaimer:** This investigation is for educational purposes. The author does not encourage harassment. All information is based on publicly available data. Subtitle Studio and Articue Tech Limited are trademarks of their respective owners.
