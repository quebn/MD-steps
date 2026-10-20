



# QR Code Generator with OpenCV

## Step 1: Python Project Setup

Initialize python virtual environment within the project.

## Step 2: Install OpenCV

Install the contrib version of OpenCV. It contains the QR code encoder, which
is not included in the regular `opencv-python` package.

```bash
python -m pip install opencv-contrib-python

```

## Step 3: Create the script and add the imports

Create a new python file, then add the imports.

```python
import cv2
```


## Step 4: Add the settings

Below the imports, define the link to encode, the output filename, and how large
each QR module should be:

```python
LINK = "https://www.google.com"
OUTPUT_FILE = "qr_code.png"
SCALE = 20  # pixels per QR module
```

The link is intentionally hardcoded, so the program does not need command-line
arguments. `SCALE = 20` makes each module 20 pixels wide.

## Step 5: Create the QR code

The `generate_qr()` function receives text to encode and returns a completed QR
image. Build it in four small parts so each operation is clear.

### Step 5.1: Define the function and its settings

```python
def generate_qr(text: str):
    params = cv2.QRCodeEncoder_Params()
    params.correction_level = cv2.QRCodeEncoder_CORRECT_LEVEL_M
```

- `def generate_qr(text: str):` declares a function that accepts the text to
  encode.
- `QRCodeEncoder_Params()` creates the settings object used by the encoder.
- `correction_level = ..._M` chooses the error-correction level. The example
  uses `M`, but you can choose any of the options below.

The error-correction level controls how much of the QR code can be damaged
while remaining readable. Higher levels add more backup data, making the code
more robust but usually larger or denser.

| Level | OpenCV constant | Approximate recovery | Effect |
| --- | --- | --- | --- |
| `L` | `cv2.QRCodeEncoder_CORRECT_LEVEL_L` | 7% | Smallest; least damage tolerance |
| `M` | `cv2.QRCodeEncoder_CORRECT_LEVEL_M` | 15% | Balanced; used in the example |
| `Q` | `cv2.QRCodeEncoder_CORRECT_LEVEL_Q` | 25% | More robust; usually larger |
| `H` | `cv2.QRCodeEncoder_CORRECT_LEVEL_H` | 30% | Most robust; usually largest |

To use a different level, change only the constant on the
`params.correction_level` line. For example:

```python
params.correction_level = cv2.QRCodeEncoder_CORRECT_LEVEL_H
```

The rest of the program stays the same. The final reference file uses `M`.

### Step 5.2: Create the encoder and encode the text

```python
    encoder = cv2.QRCodeEncoder_create(params)
    qr_image = encoder.encode(text)
```

`QRCodeEncoder_create(params)` creates the encoder using the settings from
Step 5.1. `encoder.encode(text)` converts the text into a grayscale QR image,
where black and white modules are stored as `0` and `255`. The result is stored
in `qr_image`.

### Step 5.3: Add the quiet zone

```python
    qr_image = cv2.copyMakeBorder(qr_image, 2, 2, 2, 2, cv2.BORDER_CONSTANT, value=255)
```

`copyMakeBorder()` adds space around an image. The arguments in this line are:

- `qr_image`: the source image that will receive the border.
- `2, 2, 2, 2`: the number of pixels added to the top, bottom, left, and right.
  At this stage one QR module is one pixel, so this adds two modules per side.
- `cv2.BORDER_CONSTANT`: fill the new border with one constant value instead of
  extending or reflecting the image's existing pixels.
- `value=255`: use `255` as that constant value. In a grayscale image, `255` is
  white, so the added border becomes a clean white quiet zone.

OpenCV already adds a 2-module margin, so adding 2 more on each side produces
the standard 4-module quiet zone that scanners need to detect the code's edges.

OpenCV also provides other border types, but they are not suitable for a QR
quiet zone:

| Border type | What it does | Use for QR? |
| --- | --- | --- |
| `cv2.BORDER_CONSTANT` | Fills the border with the given `value` | Yes: creates a solid white margin |
| `cv2.BORDER_REPLICATE` | Repeats the nearest edge pixels | No: copies module patterns into the margin |
| `cv2.BORDER_REFLECT` | Mirrors the image, including the edge pixels | No: creates mirrored patterns |
| `cv2.BORDER_REFLECT_101` | Mirrors the image, excluding the edge pixels; also available as `BORDER_DEFAULT` | No: creates mirrored patterns |
| `cv2.BORDER_WRAP` | Wraps pixels from the opposite side of the image | No: creates unrelated patterns |

`value=255` is used only by `BORDER_CONSTANT`. For the other border types, the
margin is generated from existing image pixels, so it will not remain solid
white.

### Step 5.4: Enlarge the image and return it

```python
    return cv2.resize(qr_image, (qr_image.shape[1] * SCALE, qr_image.shape[0] * SCALE), interpolation=cv2.INTER_NEAREST)
```

The new width and height are the number of QR modules multiplied by `SCALE`.
`INTER_NEAREST` enlarges the image without blurring the module edges. `return`
sends the finished image back to the code that called the function.

Once these four parts are added in order, `generate_qr()` is complete. Its
finished version appears in the full file at the end of these instructions.

## Step 6: Display, then save the QR code

Now create an entrypoint function called `main()`

### Step 6.1: Start the function and generate the image

```python
def main():
    running = True
    image = generate_qr(LINK)
```

- `def main():` entrypoint function
- `running = True` controls whether the preview window stays open.
- `image = generate_qr(LINK)` calls the QR function from Step 5 and stores the
  returned image in `image`.

### Step 6.2: Show the preview until the user presses `q`

```python
    while running:
        key = cv2.waitKey(1) & 0xFF
        if key == ord("q"):
            running = False

        cv2.imshow("QR code Sample", image)
```

The `while` loop keeps the preview window open. `cv2.waitKey(1)` waits briefly
for a key press, and `& 0xFF` keeps only the key code needed for comparison. If
the key is `q`, `running` becomes `False`, so the loop ends on its next check.
`cv2.imshow()` draws and refreshes the image in a window titled
**QR code Sample**.

### Step 6.3: Save the image

```python
    if not cv2.imwrite(OUTPUT_FILE, image):
        raise SystemExit(f"Could not write image to {OUTPUT_FILE}")
```

After the preview loop ends, `cv2.imwrite()` saves `image` using the filename in
`OUTPUT_FILE`. It returns `False` if saving fails, so the `if not` check stops
the program with an error message instead of silently losing the file.

### Step 6.4: Confirm that saving succeeded

```python
    print(f"QR code for {LINK} saved to {OUTPUT_FILE}")
```

This final message tells the user which link was encoded and where the image was
saved. Once these four parts are added in order, `main()` is complete.

## Step 7: Add the entry point

Finally, make the script call `main()` when it is run directly:

```python
if __name__ == "__main__":
    main()
```

This guard prevents `main()` from running if the file is imported by another
program.

## Step 8: Run and verify

Run the script from the `barcode` folder:

```bash
python qr_code.py
```

A window titled **QR code Sample** should appear. Press `q` to close it and save
the image. Then:

1. Confirm that `qr_code.png` exists in the same folder.
2. Open the image or scan it with a phone.
3. Check that the scanned link is `https://www.google.com`.

## Optional: Use a different link

Change the value of `LINK` near the top of the file and run the program again.
The new QR code will overwrite `qr_code.png`.
