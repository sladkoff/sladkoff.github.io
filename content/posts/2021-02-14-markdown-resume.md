---
title: How I build my resume from Markdown
date: "2021-02-15"
comments: true
draft: true
tags:
  - markdown
  - pdf
  - github
  - opensource
  - resume
---

I was recently inspired to update my Software Engineering resume. 
The problem with that was that up until now my resume was a Word `docx` document that I would manually export as PDF and distribute.
I used to always get frustrated when I had to touch this document because I **hate** formatting text in Word and/or Google Docs.
 
After stumbling upon this [Techlead video on YouTube](https://www.youtube.com/watch?v=xpaz7nrNmXA) I was convinced that I didn't
need any fancy layouts for my CV anyway and that I would go with a minimalist plain-text approach.

If you're like me and you don't need pie charts on your resume, you might be asking:

> Why not write my resume in good old Markdown? Why not just make it open-source on Github as well? 
Why not use Github Actions to build a distributable PDF version of it? :thinking:

Hey, that's what I did! And I find it pretty cool. So I recommend you do it, too. Here's why.

## The Opensource Markdown Resume :tm:

### Easy to write and maintain!

It's one file. It's simple **af**. I only used headings and bullet lists and decided that it was enough to get my points across.

You can see it on [Github](https://github.com/sladkoff/resume/edit/master/README.md) and fork it or whatever.

### Automated PDF export!

I found this [markdown-to-pdf](https://github.com/BaileyJM02/markdown-to-pdf) Github Action that exports - you guessed it - Markdown to PDF. 
It renders the PDF with the default Github styling, so the result ([preview available here](https://github.com/sladkoff/resume/releases/tag/2021-02-13)) looks very similar to the [web version](https://github.com/sladkoff/resume).

Here's the full annotated [Github Action](https://raw.githubusercontent.com/sladkoff/resume/master/.github/workflows/release.yml) that I'm using now:

```yaml
on:
  push:
    tags:
      - '*'                                                 # (1)

name: Upload Release Asset

jobs:
  build:
    name: Upload Release Asset
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v2
      - name: Remove swag line                              # (2)
        run: sed -i '1d' README.md
      - name: Build PDF from Markdown                       # (3)
        uses: BaileyJM02/markdown-to-pdf@v1
        with:
          input_dir: .
          output_dir: out
      - name: Rename pdf
        run: cp out/README.pdf ./leonid_koftun_resume.pdf
      - name: Create Release                                # (4)
        id: create_release
        uses: actions/create-release@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          tag_name: ${{ github.ref }}
          release_name: Release ${{ github.ref }}
          draft: false
          prerelease: false
      - name: Upload Release Asset                          # (5)
        id: upload-release-asset
        uses: actions/upload-release-asset@v1
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
        with:
          upload_url: ${{ steps.create_release.outputs.upload_url }}
          asset_path: ./leonid_koftun_resume.pdf
          asset_name: leonid_koftun_resume.pdf
          asset_content_type: application/pdf
```

#### Annotations

1. Build PDF when any tag is pushed
2. (Optional) This is just 'pre-processing' to remove some HTML content that I don't want to be rendered to PDF (the first line of the Markdown contains some links that only make sense on the Github web view)
3. The actual [markdown-to-pdf](https://github.com/BaileyJM02/markdown-to-pdf) step with minimal config
4. Creates a Github release for the tag
5. Uploads the generated PDF to the tagged release

### The opensource aspect

As you probably noticed by now, my resume is publicly available on Github.

Potential employers and recruiters can grab a copy there. My peers and colleagues can review it and tell me what I suck at the most (come at me).

### Any issues with this?

Yes.

- If you want to use fancy column layouts and colors and unicorns :unicorn: in your resume, then this probably isn't for you (you could make it complicated and add CSS and templating but that kind of defeats the purpose of simplicity in the `.md` approach IMO).
- If you're afraid of me or one of my cats copying some lines from your own personal resume, I do not recommend that you make it public.

## Conclusion

> Keep your resume simple by using Markdown, publish it on Github if you want to be a cool kid.
 
What do you think of this nonsense? #hmu on [Twitter](https://twitter.com/sladkovik) or [Instagram](https://www.instagram.com/sladkoff2/). :call_me_hand: 
