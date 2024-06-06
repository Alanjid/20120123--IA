```
import cv2 as cv 
cap=cv.VideoCapture(0)
while(True):
    res,img=cap.read()
    img2 = cv.cvtColor(img, cv.COLOR_BGR2HSV)
    ubb = (40,60,60)
    uba =(60, 255,255)
    mask=cv.inRange(img2,ubb,uba)
    res=cv.bitwise_and(img,img,mask=mask)
    cv.imshow('captura',mask)
    if cv.waitKey(1) & 0xFF == ord('s'):
        break
       
cap.release()
cv.destroyAlllWindows()
```


    ---------------------------------------------------------------------------

    AttributeError                            Traceback (most recent call last)

    Cell In[4], line 15
         12         break
         14 cap.release()
    ---> 15 cv.destroyAlllWindows()
    

    AttributeError: module 'cv2' has no attribute 'destroyAlllWindows'



```
import numpy as np
import cv2 as cv
import math 

rostro = cv.CascadeClassifier('C:\\Users\\alanj\\haarcascade_frontalface_alt.xml')
cap = cv.VideoCapture(0)
while True:
    ret, frame = cap.read()
    gray = cv.cvtColor(frame, cv.COLOR_BGR2GRAY)
    rostros = rostro.detectMultiScale(gray, 1.3, 5)
    for(x, y, w, h) in rostros:
        frame = cv.rectangle(frame, (x,y), (x+w, y+h), (255, 0, 0), 5)
    cv.imshow('rostros', frame)
    k = cv.waitKey(1)
    if k == 27:
        break
cap.release()
cv.destroyAllWindows()
```


```

```
