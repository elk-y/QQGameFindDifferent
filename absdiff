import sys

import win32gui

import cv2
from PyQt5.QtCore import *
from PyQt5.QtGui import *
from PyQt5.QtWidgets import *
import numpy as np

class Gui(QWidget):
    def __init__(self):
        QWidget.__init__(self)
        self.setWindowTitle('Tool')
        explain = QLabel("操作说明：保持游戏窗口在屏幕内，点击找茬获取结果。")
        self.button = QPushButton("找茬")
        vbox = QVBoxLayout()
        vbox.addWidget(explain)
        vbox.addWidget(self.button)
        self.image = QLabel("结果展示")
        self.image.setAlignment(Qt.AlignCenter)
        vbox.addWidget(self.image)
        self.setLayout(vbox)
        self.button.clicked.connect(self.find_diff)


    def find_diff(self):
        try:
            self.button.setText("找茬")
            img = self.get_screenshot()
            diff_img = self.get_diff(img)
            pix = self.cvimg_to_qtimg(diff_img)
            self.image.setPixmap(QPixmap.fromImage(pix))
        except:
            self.button.setText("找茬（未找到游戏图片）")
            return

    # 获取屏幕截图
    def get_screenshot(self):
        hwnd_title = dict()
        def get_all_hwnd(hwnd, mouse):
            if win32gui.IsWindow(hwnd) and win32gui.IsWindowEnabled(hwnd) and win32gui.IsWindowVisible(hwnd):
                hwnd_title.update({hwnd: win32gui.GetWindowText(hwnd)})
        win32gui.EnumWindows(get_all_hwnd, 0)
        hwnd = win32gui.FindWindow(None, '大家来找茬')
        screen = QApplication.primaryScreen()
        img = screen.grabWindow(hwnd).toImage()
        image = self.QImage2Mat(img)
        return image


    def get_diff(self,img):
        h_img,w_img = img.shape[:2]
        area = h_img * w_img
        imgray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
        ret, thresh = cv2.threshold(imgray, 127, 255, 0)
        contours, hierarchy = cv2.findContours(thresh, cv2.RETR_TREE, cv2.CHAIN_APPROX_SIMPLE)
        #cv2.drawContours(img, contours, -1, (0, 0, 255), 3) # 绘制轮廓
        # 找到图片区
        rois = []
        for cnt in contours:
            x, y, w, h = cv2.boundingRect(cnt)
            if area//7 < w * h < area//6.5:
                rois.append([x + 10, y + 10, w - 20, h - 20])
        # 利用absdiff函数找不同
        # 找到区域内面积相等的两个框为图片取样区
        for i in range(len(rois)):
            for j in range(i + 1, len(rois)):
                if rois[i][2] * rois[i][3] == rois[j][2] * rois[j][3]:
                    roi1 = img[rois[i][1]:rois[i][1]+rois[i][3],rois[i][0]:rois[i][0]+rois[i][2]]
                    roi2 = img[rois[j][1]:rois[j][1]+rois[j][3],rois[j][0]:rois[j][0]+rois[j][2]]
        roi = cv2.absdiff(roi1, roi2)  #对比查找图片不同之处

        # 对差异区域加强调整
        frame = self.adjust(roi, 1.3, 0)
        cls = self.closing(frame, 5)
        roigray = cv2.cvtColor(cls, cv2.COLOR_BGR2GRAY)
        ret, thresh = cv2.threshold(roigray, 30, 255, 0)
        thresh = self.closing(thresh, 6)
        contours, hierarchy = cv2.findContours(thresh, cv2.RETR_TREE, cv2.CHAIN_APPROX_SIMPLE)

        # 框出不同之处
        for cnt in contours:
            x, y, w, h = cv2.boundingRect(cnt)
            cv2.rectangle(roi1, (x, y), (x + w, y + h), (0, 255, 0), 3)
        return roi1

    # 将opencv图像转为QImage
    def cvimg_to_qtimg(self, img):
        img = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
        x = img.shape[1]
        y = img.shape[0]
        frame = QImage(img.data, x, y, x * 3, QImage.Format_RGB888)
        return frame

    # QImage转opencv图像
    def QImage2Mat(self,incomingImage):
        incomingImage = incomingImage.convertToFormat(4)
        width = incomingImage.width()
        height = incomingImage.height()
        ptr = incomingImage.bits()
        ptr.setsize(incomingImage.byteCount())
        arr = np.array(ptr).reshape(height, width, 4)  # Copies the data
        return arr

    # 增强图片对比度，亮度
    def adjust(self, img, c, b):
        h, w, r = img.shape
        blank = np.zeros([h, w, r], img.dtype)
        dst = cv2.addWeighted(img, c, blank, 1 - c, b)
        return dst

    # 图片闭运算，将轮廓填充成实心
    def closing(self, frame, n):
        kernel = cv2.getStructuringElement(cv2.MORPH_ELLIPSE, (n, n))
        closing = cv2.morphologyEx(frame, cv2.MORPH_CLOSE, kernel)
        kernel = cv2.getStructuringElement(cv2.MORPH_RECT, (n, n))
        closing = cv2.morphologyEx(closing, cv2.MORPH_CLOSE, kernel)
        return closing


app = QApplication(sys.argv)
qb = Gui()
qb.show()
sys.exit(app.exec_())
