---
id: skill-image-vision
type: skills
title: "Image Vision Skill"
---

# Image Vision Skill

Analyze images using LLM vision APIs.

## Overview

The Image Vision skill teaches Amplifier how to:

- Understand image content
- Describe visual elements
- Answer questions about images
- Compare multiple images
- Extract text (OCR)

## Loading the Skill

```
> Load the image-vision skill
```

Once loaded, Amplifier knows how to work with images.

## When to Use

| Task | Use Image Vision |
|------|------------------|
| Describe what's in an image | ✅ |
| Extract text from screenshot | ✅ |
| Compare two images | ✅ |
| Answer questions about image | ✅ |
| Edit or modify images | ❌ Different tools |
| Generate images | ❌ Different tools |

## Supported Providers

| Provider | Model | Vision Capable |
|----------|-------|----------------|
| Anthropic | Claude 3+ | ✅ |
| OpenAI | GPT-4V, GPT-4o | ✅ |
| Google | Gemini Pro Vision | ✅ |
| Azure OpenAI | GPT-4V | ✅ |

## Core Patterns

### Describe Image

```
> What's in this image?
[attach: screenshot.png]
```

### Extract Text (OCR)

```
> Extract all text from this image
[attach: document-scan.png]
```

### Answer Questions

```
> How many people are in this photo?
[attach: group-photo.jpg]
```

### Compare Images

```
> What's different between these two screenshots?
[attach: before.png]
[attach: after.png]
```

## Image Sources

### Local Files

```
> Analyze the image at ./screenshots/error.png
```

### URLs

```
> Describe the image at https://example.com/photo.jpg
```

### Base64

For programmatic use:

```python
import base64

with open("image.png", "rb") as f:
    image_data = base64.b64encode(f.read()).decode()

# Include in prompt with data URI
```

## Best Practices

### Be Specific

```
# Vague
> What's this?

# Specific
> What error message is shown in this screenshot?
```

### Provide Context

```
> This is a screenshot of our checkout page.
> Is the "Complete Purchase" button visible?
```

### Ask Focused Questions

```
# Too broad
> Tell me everything about this image

# Focused
> What color is the submit button in this form?
```

## Common Tasks

### UI Validation

```
> Does this screenshot show a successful login?
> Is the error message displayed in red?
> Are all form fields filled in?
```

### Content Understanding

```
> What products are shown in this catalog image?
> Summarize the key points from this infographic
> What's the main headline on this webpage?
```

### Data Extraction

```
> Extract the table data from this spreadsheet screenshot
> What are the values in the pie chart?
> Read the receipt amounts from this photo
```

### Comparison

```
> Has the layout changed between these two versions?
> What elements are missing in the second screenshot?
> Which design looks more professional?
```

## Limitations

### Cannot Do

- Edit or modify images
- Generate new images
- Process video (frame by frame only)
- Guarantee 100% accurate OCR

### Image Size

Large images may be resized. For text extraction:
- Use highest resolution available
- Ensure text is readable at smaller sizes

### Complex Documents

For complex documents with many elements:
- Ask about specific sections
- Break into multiple queries

## Try It Yourself

### Exercise 1: Describe Image

```
> Load image-vision skill
> Describe what you see at https://picsum.photos/800/600
```

### Exercise 2: Extract Text

Take a screenshot of any webpage and ask:
```
> Extract the main heading from this screenshot
```

### Exercise 3: Compare

Take two screenshots and ask:
```
> What changed between these two screenshots?
```

## Provider Configuration

Ensure your provider supports vision:

```yaml
providers:
  - module: provider-anthropic
    config:
      model: claude-sonnet-4-20250514  # Supports vision
```

## Troubleshooting

### "Vision not supported"

- Check model supports vision
- Update to vision-capable model

### "Image too large"

- Reduce image resolution
- Crop to relevant area

### "Cannot read text"

- Improve image quality
- Increase contrast
- Try different angle/lighting

## Source

```
robotdad/skills/image-vision/
├── SKILL.md     # Core workflow
├── patterns.md  # Advanced patterns
└── setup.md     # Provider setup
```
