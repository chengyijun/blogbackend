---
layout: pyqt5
title: stacked组件和tab组件的使用
date: 2022-07-15 11:12:08
tags:
---
~~~python


# -*- coding: utf-8 -*-
# @Author  : chengyijun
# @Time    : 2020/9/17 15:21
# @File    : stacked_demo.py
# @desc    : pyqt5 stacked组件和tab组件的使用
import sys

from PyQt5.QtCore import pyqtSlot
from PyQt5.QtWidgets import QApplication, QMainWindow, QWidget, QLabel

from login import Ui_Form as Ui_Login
from tab import Ui_Form as Ui_Tab
from stacked import Ui_MainWindow


class Tab(Ui_Tab, QWidget):

    def __init__(self, parent=None) -> None:
        super().__init__(parent=parent)
        self.setupUi(self)

        q_label = QLabel(text="我是tab的内容")
        self.tabWidget.addTab(q_label, "tab3")


class Login(Ui_Login, QWidget):

    def __init__(self, parent=None) -> None:
        super().__init__(parent=parent)
        self.setupUi(self)


class Demo(Ui_MainWindow, QMainWindow):

    def __init__(self) -> None:
        super().__init__()
        self.setupUi(self)

    @pyqtSlot()
    def on_pushButton_clicked(self):
        # self.stackedWidget.setCurrentIndex(0)
        login = Login()
        self.stackedWidget.insertWidget(0, login)
        self.stackedWidget.setCurrentIndex(0)

    @pyqtSlot()
    def on_pushButton_2_clicked(self):
        tab = Tab()
        self.stackedWidget.insertWidget(1, tab)
        self.stackedWidget.setCurrentIndex(1)

    @pyqtSlot()
    def on_pushButton_3_clicked(self):
        print('gwc')


def main():
    app = QApplication(sys.argv)
    demo = Demo()
    demo.show()
    sys.exit(app.exec_())


if __name__ == '__main__':
    main()



~~~
