pip install opencv-python numpy PyOpenGL
import cv2
import numpy as np
from OpenGL.GL import *
from OpenGL.GLUT import *
from OpenGL.GLU import *

# Global variable for capturing video
cap = None

def init_camera():
    global cap
    cap = cv2.VideoCapture(0)  # Open the default camera

def release_camera():
    global cap
    if cap is not None:
        cap.release()

def cartoonize_image(img):
    # Convert to gray scale
    gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
    # Apply median filter
    gray = cv2.medianBlur(gray, 7)
    # Detect edges
    edges = cv2.adaptiveThreshold(gray, 255, cv2.ADAPTIVE_THRESH_MEAN_C, cv2.THRESH_BINARY, 9, 2)
    # Apply bilateral filter to get cartoon effect
    color = cv2.bilateralFilter(img, 9, 250, 250)
    # Combine edges and color
    cartoon = cv2.bitwise_and(color, color, mask=edges)
    return cartoon

def display():
    global cap
    ret, frame = cap.read()  # Capture frame-by-frame
    if not ret:
        return
    
    cartoon_frame = cartoonize_image(frame)
    height, width, channels = cartoon_frame.shape
    glClear(GL_COLOR_BUFFER_BIT)
    glDrawPixels(width, height, GL_BGR, GL_UNSIGNED_BYTE, cartoon_frame)
    glutSwapBuffers()

def main():
    glutInit()
    glutInitDisplayMode(GLUT_RGBA | GLUT_DOUBLE | GLUT_ALPHA | GLUT_DEPTH)
    glutInitWindowSize(640, 480)
    glutInitWindowPosition(0, 0)
    glutCreateWindow(b"2D Animation Camera")
    glutDisplayFunc(display)
    glutIdleFunc(display)
    init_camera()
    glutMainLoop()
    release_camera()

if __name__ == "__main__":
    main()
