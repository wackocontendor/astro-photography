# astro-photography
Not quite what it says on the tin - I built a simple website using the Astro framework to show off some of my photography.

GitHub Copilot wrote the code, I took the photos though.

Click [here](https://louisandalens.pictures) to check it out.

For those that are interested, the JPEGs were processed with Pillow. Their EXIF orientation was applied directly to the image pixels using `ImageOps.exif_transpose`, then each file was converted to RGB and re-saved without EXIF or XMP metadata. `exiftool` was used to read the original focal length, aperture, shutter speed, and capture date, which were then transcribed into the corresponding photo entries in `src/pages/index.astro`.

## Processing new photos

The following workflow makes the image preparation reproducible without an AI assistant:

1. Install the prerequisites:

   Install Python 3, Node.js 22.23.4 or newer, and [`exiftool`](https://exiftool.org/). Node.js/npm is used to run Astro; Python is used for Pillow image processing.

   ```sh
   python3 -m pip install Pillow
   ```

2. Copy the original JPEGs into `public/images/`. Keep a backup because the processing script below replaces each file.

3. Inspect the camera metadata before removing it:

   ```sh
   exiftool -s -s -s -FileName -FocalLength -FNumber \
     -ExposureTime -DateTimeOriginal -Orientation public/images/*.JPG
   ```

   Use `FocalLength`, `FNumber`, `ExposureTime`, and `DateTimeOriginal` to create entries in the `photos` array in `src/pages/index.astro`, for example:

   ```ts
   { file: "IMG_0001.JPG", focalLength: "35mm", aperture: "f/2.8", shutter: "1/250s", date: "Nov '26" },
   ```

4. Apply the EXIF orientation to the actual pixels and remove the metadata with Pillow:

   ```python
   from pathlib import Path
   from PIL import Image, ImageOps

   for path in Path("public/images").glob("*.JPG"):
       with Image.open(path) as source:
           image = ImageOps.exif_transpose(source).convert("RGB")
           image.save(path, format="JPEG", quality=95, optimize=True)
   ```

   `ImageOps.exif_transpose` handles rotation and mirroring from the EXIF orientation tag. Saving a new RGB JPEG without passing EXIF data removes the original EXIF metadata.

5. Confirm that the metadata was removed and build the site:

   ```sh
   exiftool -s -Orientation -FocalLength -GPSLatitude public/images/IMG_0001.JPG
   npm run build
   ```