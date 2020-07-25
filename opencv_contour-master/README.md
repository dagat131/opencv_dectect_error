# opencv_contour
- opencv를 이용하여 회로기판의 접점 결함을 찾아내는 코드입니다.
- opencv Repository에서 배웠던 코드를 응용하여 구현해 보았습니다.

## 1. 결과물
- 회로기판에서 세군데의 접점 영역별로 양품이면 녹색, 불량이면 빨간색으로 표시하고<br>
  전체적으로 불량이 없을 경우, "OK" 표기를 통해 양품/불량 판정하게 됩니다.
- 양품/불량의 구분은 접점의 면적이 기준치 이상인지를 비교하게 로직을 구현하였습니다.<br><br>

실행결과<br>
<img src="/images/cv_contour_output.jpg"><br><br>

## 2. 코드구현

```python
import cv2
import numpy as np

src = cv2.imread("images/13_NG.bmp", cv2.IMREAD_COLOR)
src = cv2.pyrDown(src)
src_2 = src.copy()

gray2 = cv2.cvtColor(src_2, cv2.COLOR_BGR2GRAY)
ret2, binary_2 = cv2.threshold(gray2, 100,255,cv2.THRESH_BINARY)
kernel = np.ones((7,7), np.uint8)
ng_cnt = 0 #전체 ng개수 파악하여 OK, NG 구분

#contour 파악을 위한 함수 선언===============================================================
def contour(img_src, high):
    global ng_cnt #전역변수를 수정하기 위한 global 선언

    img_src = cv2.morphologyEx(img_src, cv2.MORPH_CLOSE, kernel)
    contours, hierarchy = cv2.findContours(img_src, cv2.RETR_CCOMP, cv2.CHAIN_APPROX_NONE)

    mu = [None]*len(contours)
    mc = [None]*len(contours)

    ng = 0

    for i in range(len(contours)) :
        mu[i] = cv2.moments(contours[i])#중심점
        mc[i] = (mu[i]['m10'] / (mu[i]['m00'] + 1e-5), mu[i]['m01'] / (mu[i]['m00'] + 1e-5))
        c_area = cv2.contourArea(contours[i]) #contour의 면적

        if c_area > 2000 and c_area < high : #네모는 찾아내고, 원은 찾아내지 않게끔 면적 범위 설정
            ng += 1 # 영역별 ng횟수 증가
            ng_cnt += 1 #전체 영역 ng횟수 증가
            
    if ng > 0 : #영역별 ng횟수가 1이상이면 해당 영역은 모두 붉은색으로 그림
        color= (0, 0, 255) 
    else: #영역별 ng횟수가 0이면 해당 영역은 모두 녹색으로 그림
        color = (0, 255, 0)
    draw_contour(contours, img_copy, color) #draw_contour함수 실행
#==========================================================================================

#contour를 그리기위한 함수 선언===============================================================
def draw_contour(contours, img_copy, color):
    for i in range(len(contours)) :
        c_area = cv2.contourArea(contours[i])
        
        if c_area > 2000: # 원 영역 면적보다 클 경우에만 그림
            cv2.drawContours(img_copy, contours, i, color, 2)
            cv2.putText(img_copy,"{} : {}".format(i,c_area) ,tuple(contours[i][0][0]),
                        cv2.FONT_HERSHEY_COMPLEX,0.4, color,1)
#==========================================================================================


#실행코드==========================================================================
img_area = [206,481, 35,205, 394,522, 210,1119, 241,464, 1149,1291] 
#좌측, 중앙, 우측의 면적을 각각 리스트 안에 순서대로 넣어줌(for문으로 돌리기 위함)

j = 0 #1이면 좌측, 2면 중앙, 3이면 우측

#for i in range(시작값, 끝값(-1), 증가값)
for i in range(0,len(img_area),4):

    j += 1
    img_src = binary_2[img_area[i]:img_area[i+1], img_area[i+2]:img_area[i+3]]
    img_copy = src[img_area[i]:img_area[i+1], img_area[i+2]:img_area[i+3]]
    
    if j == 1 or j == 3:
        high = 3700
    else:
        high = 3500

    contour(img_src, high) #contour 함수 실행

    k = (j-1) * 4 #변수 j에서 1씩 뺀 값에 4를 곱하기 함(그래야 기존 반복문에 img_area 리스트 안의 값을 가져올수 있음)
    src_2[img_area[k]:img_area[1+k], img_area[2+k]:img_area[3+k]] = img_copy

cv2.rectangle(src_2,(28,183) ,(200,491) ,(0,255,0),2)
cv2.rectangle(src_2,(210,396) ,(1105,500) ,(0,255,0),2)
cv2.rectangle(src_2,(1135,183) ,(1270,491) ,(0,255,0),2)    

if ng_cnt == 0: #전체 ng갯수가 0이면 OK판정함
    cv2.putText(src_2,"OK",(452,632), cv2.FONT_HERSHEY_COMPLEX,3, (0,255,0),1)
else:
    cv2.putText(src_2,"NG",(452,632), cv2.FONT_HERSHEY_COMPLEX,3, (0,0,255),1)
 
cv2.imshow("src2",src_2)
cv2.waitKey(0)

cv2.destroyAllWindows()
```
- opencv의 contour함수를 이용하여 외곽선과 중심값, 면적을 구하게 됩니다.
- 영역별로 면적을 구하고, 외곽선을 그리는 동작은 반복되기에 함수로 묶었습니다.
<br><br>

## 3. 추가로 구현해야될 부분
- 추후 image파일을 선택할수 있게끔 포맷 변경 예정입니다.
- 각 영역별로 접점의 번호 부여가 제각각입니다. 중심점의 위치값을 기준으로 정렬하여 부여 예정입니다.
- PyQT를 이용한 GUI로 수정할 예정입니다.
- 위 사항이 수정되면 본 README.md 하단에 추가 작성하도록 하겠습니다.




