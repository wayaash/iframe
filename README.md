from PIL import Image, ImageDraw, ImageFilter, ImageFont
import numpy as np

W, H = 800, 800
img = Image.new("RGBA", (W, H), (255, 255, 255, 255))
draw = ImageDraw.Draw(img)

samurai = [
    (400, 150), (450, 240), (480, 480), (450, 600),
    (400, 700), (350, 600), (320, 480), (350, 240)
]

def vertical_gradient(size, top_color, bottom_color):
    w, h = size
    gradient = np.zeros((h, w, 3), dtype=np.uint8)
    for y in range(h):
        ratio = y / (h - 1)
        gradient[y, :, :] = [
            int(top_color[i] * (1 - ratio) + bottom_color[i] * ratio)
            for i in range(3)
        ]
    return Image.fromarray(gradient)

bg = vertical_gradient((W, H), (30, 30, 60), (200, 80, 80))
img = Image.alpha_composite(bg.convert("RGBA"), img)

mask = Image.new("L", (W, H), 0)
ImageDraw.Draw(mask).polygon(samurai, fill=255)
rad = np.linspace(0, 1, W//2)
rad = np.clip(1 - rad, 0, 1)
rg = np.tile(rad[:, None], (1, H)).T * 255
radial = Image.fromarray(rg.astype(np.uint8), mode="L").resize((W, H))
samurai_colored = Image.new("RGBA", (W, H), (0, 0, 0, 0))
samurai_colored.putalpha(radial)
img = Image.composite(samurai_colored, img, mask)

draw = ImageDraw.Draw(img)

try:
    font = ImageFont.truetype("arial.ttf", 48)
except IOError:
    font = ImageFont.load_default()
text = "Yash Raghuwanshi"
bbox = draw.textbbox((0, 0), text, font=font)
text_w, text_h = bbox[2] - bbox[0], bbox[3] - bbox[1]
draw.text(((W - text_w) // 2, H - text_h - 50), text, fill="white", font=font)

img = img.filter(ImageFilter.GaussianBlur(2)).filter(ImageFilter.EDGE_ENHANCE_MORE)

img.save("samurai_art.png")
print("Saved as samurai_art.png")
