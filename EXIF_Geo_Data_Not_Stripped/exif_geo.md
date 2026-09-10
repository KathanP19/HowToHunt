# EXIF Geolocation Data Not Stripped

When users upload images (avatars, attachments, reports), many applications never strip the **EXIF metadata** embedded by phones and cameras — GPS coordinates, device make/model, software, timestamps. Anyone who can fetch the stored image can then extract other users' physical locations and device fingerprints. This is a privacy/PII disclosure bug; it has been accepted as P1 at some programs (see report below).

## Hunting Checklist

1. Find every image/file upload feature: profile pictures, attachments, chat, support forms, report flows.
2. Prepare sample images that **carry EXIF data** (the exif-samples repo below) or set GPS fields on your own photos.
3. Upload as user A, retrieve the stored file URL (copy image address, or inspect the `<img>` tag for the URL).
4. Fetch the stored/processed image and check whether metadata survived **processing** (resize/re-encode usually strips it — that's the bug when it doesn't).
5. Repeat across formats (JPG/PNG/TIFF) and sizes; some code paths bypass the sanitizer.
6. Verify with a second reader (`exiftool`) and report with the extracted fields redacted where sensitive.

## Test Cases & Examples

Original steps (kept verbatim in spirit):

1. Go to GitHub: https://github.com/ianare/exif-samples/tree/master/jpg — it has many images with GPS/EXIF data at different resolutions (e.g. 1280×720) and file sizes.
2. Go to the upload option on the target website.
3. Upload the image.
4. Get the stored path of the uploaded image (right-click → *Copy image address*, or right-click → *Inspect*, the URL appears in the `<img src>` — *Edit as HTML* to copy it).
5. Open the stored image at an EXIF reader, e.g. https://www.regex.info/exif.cgi (formerly http://exif.regex.info/exif.cgi).
6. If EXIF is still shown on the stored file — report it.

Local verification commands:

```bash
# Read metadata from the stored copy (the one served by the target)
curl -sk https://target.com/uploads/avatars/1234.jpg -o stored.jpg
exiftool stored.jpg          # GPSLatitude/GPSLongitude, Make, Model, Software...

# Quick check for GPS tags only
exiftool -gps:all stored.jpg

# Create a GPS-bearing test image (or grab one from ianare/exif-samples)
exiftool -GPSLatitude=37.77 -GPSLongitude=-122.42 -overwrite_original test.jpg
```

## Bypass Techniques

- **Different code paths strip differently:** the thumbnail pipeline may preserve EXIF while the full-size path strips it (or vice-versa) — download *every* variant the site exposes.
- **Direct S3/CDN URLs vs re-served content:** sometimes the app strips on re-serve but the origin bucket keeps the original with metadata.
- **Other metadata sinks:** some uploaders leak in sidecar files, XMP blocks in PNG (tEXt chunks), or TIFF/HEIC which naive "image processors" pass through untouched.

## Tools & Resources

- [exif-samples](https://github.com/ianare/exif-samples) — ready-made JPG/PNG samples carrying EXIF/GPS data (as in the original method)
- [exiftool](https://exiftool.org) — read/write EXIF from CLI
- Online EXIF viewers: [exif.regex.info](http://exif.regex.info/exif.cgi) (as in the original method), [metapicz](http://metapicz.com)
- [pyexifinfo / Piexif](https://github.com/h2non/Piexif) — scriptable stripping/inspection for bulk testing

## Reports (HackerOne)

- [IDOR with Geolocation data not stripped from images](https://hackerone.com/reports/906907)

## Report Tips

- Frame as **PII/location disclosure of other users**, not "metadata exists": show user A's exact GPS coords retrievable by user B/anonymous visitors.
- The linked report shows the classic pairing: missing strip + IDOR = high/critical; strip-only issues often land as medium — note which variant you proved.
- Recommend: strip EXIF on upload (or re-encode images) and fix access control on stored files.

## Author

- [@0xd3vil](https://twitter.com/0xd3vil)
- [@klaus](https://twitter.com/klaus_dev)
