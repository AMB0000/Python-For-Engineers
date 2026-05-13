## Chocolate Counter 

### Final Code 


import cv2

import numpy as np

import matplotlib.pyplot as plt

img = cv2.imread('img_3.jpg')
img_rgb = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
hsv = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)

mask = cv2.inRange(hsv, np.array([0, 50, 20]), np.array([20, 255, 100]))
contours, _ = cv2.findContours(mask, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_SIMPLE)

def is_chocolate(cnt):
    area = cv2.contourArea(cnt)
    if not (500 <= area <= 6000): return False
    x, y, w, h = cv2.boundingRect(cnt)
    return min(w, h) / max(w, h) > 0.4

chocolates = [cnt for cnt in contours if is_chocolate(cnt)]
output = img_rgb.copy()
for i, cnt in enumerate(chocolates):
    M = cv2.moments(cnt)
    cx, cy = int(M["m10"]/M["m00"]), int(M["m01"]/M["m00"])
    cv2.drawContours(output, [cnt], -1, (0,255,0), 2)
    cv2.circle(output, (cx,cy), 6, (255,0,0), -1)
    cv2.putText(output, str(i+1), (cx-8, cy-12), cv2.FONT_HERSHEY_SIMPLEX, 0.7, (255,255,255), 2)

plt.imshow(output)
plt.title(f'Detected Chocolates: {len(chocolates)}')
plt.axis('off')
plt.show()

print(f'Total chocolates detected: {len(chocolates)}')


### Output

![Alt text]([https://link-to-your-image.com/image.png](https://github.com/AMB0000/Python-For-Engineers/blob/b9e2bab2eee493fb3fcc223f1db3494f64d58426/LAB06/CHOCO_FINAL.png))

