# python-06
from PIL import Image  # 用于打开图片和对图片处理
from selenium import webdriver  # 用于打开网站
import time  # 代码运行停顿
from aip import AipOcr


class VerificationCode:
    def __init__(self):
        self.driver = webdriver.Firefox()
        self.find_element = self.driver.find_element_by_css_selector

    def get_pictures(self):
        self.driver.get('https://www.cndns.com/members/signin.aspx')
        time.sleep(2)
        self.driver.save_screenshot('pictures.png')  # 全屏截图
        page_snap_obj = Image.open('pictures.png')
        img = self.find_element('img[id="VcodeImg"]')  # 验证码元素位置
        time.sleep(2)
        location = img.location
        size = img.size  # 获取验证码的大小参数
        left = location['x'] + 1550
        top = location['y'] + 740
        right = left + size['width'] + 145
        bottom = top + size['height'] + 50
        image_obj = page_snap_obj.crop((left, top, right, bottom))  # 按照验证码的长宽，切割验证码
        image_obj.show()  # 打开切割后的完整验证码
        image_obj.save("验证码.png")
        self.driver.close()  # 处理完验证码后关闭浏览器
        return image_obj

    def processing_image(self):
        image_obj = self.get_pictures()  # 获取验证码
        img = image_obj.convert("L")  # 转灰度
        pixdata = img.load()
        w, h = img.size
        threshold = 160
        # 遍历所有像素，大于阈值的为黑色
        for y in range(h):
            for x in range(w):
                if pixdata[x, y] < threshold:
                    pixdata[x, y] = 0
                else:
                    pixdata[x, y] = 255
        img.show()
        return img

    def delete_spot(self):
        images = self.processing_image()
        data = images.getdata()
        w, h = images.size
        black_point = 0
        for x in range(1, w - 1):
            for y in range(1, h - 1):
                mid_pixel = data[w * y + x]  # 中央像素点像素值
                if mid_pixel < 50:  # 找出上下左右四个方向像素点像素值
                    top_pixel = data[w * (y - 1) + x]
                    left_pixel = data[w * y + (x - 1)]
                    down_pixel = data[w * (y + 1) + x]
                    right_pixel = data[w * y + (x + 1)]
                    # 判断上下左右的黑色像素点总个数
                    if top_pixel < 10:
                        black_point += 1
                    if left_pixel < 10:
                        black_point += 1
                    if down_pixel < 10:
                        black_point += 1
                    if right_pixel < 10:
                        black_point += 1
                    if black_point < 1:
                        images.putpixel((x, y), 255)
                    black_point = 0
        # images.show()
        images.show()
        return images

    def image_str(self):
        image = self.delete_spot()
        # 下面3个变量请自行更改
        APP_ID = '19843045'
        API_KEY = '2G7SqCS8rqUGhx0juidUOw8X'
        SECRET_KEY = 'iQvzi6wEMYM8pIyWwST78GPpxA1hvLEC'

        aipOcr = AipOcr(APP_ID, API_KEY, SECRET_KEY)

        filePath = "C:/Users/14434/Desktop/识别验证码/验证码.png"

        def get_file_content(filePath):
            with open(filePath, 'rb') as fp:
                return fp.read()

        # 定义参数变量
        options = {
            'detect_direction': 'true',
            'language_type': 'CHN_ENG',
        }

        # 调用通用文字识别接口
        result = aipOcr.basicAccurate(get_file_content(filePath), options)
        a=result['words_result'][0]['words']
        print(a)


if __name__ == '__main__':
    a = VerificationCode()
    a.image_str()
