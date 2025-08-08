# Kahoot-Automated-Quiz-taker
Takes picture of current screen (Kahoot quiz screen) and automatically answers your kahoot in terminal using openais api (Model can be whatever you like)
Coordinates vary for different screens
Inspiration of proj from coding sloth!


```python
# IMPORTS ETC
import os
os.chdir(r"C:\Users\pc\Desktop\PYTHON\AI STUFF AND BOTS")

import openai
from openai import OpenAI
import numpy as np
import pytesseract
import cv2 
import pyautogui
from time import sleep
import keyboard

#FUNCTIONS
def pre_proccess_for_OCR(image_path):
    image = cv2.imread(image_path)
    image_gray = cv2.cvtColor(image, cv2.COLOR_BGR2GRAY)
    image_black = cv2.adaptiveThreshold(image_gray, 255, cv2.ADAPTIVE_THRESH_GAUSSIAN_C, cv2.THRESH_BINARY, 11, 2)
    # contours, _ = cv2.findContours(image_black, cv2.RETR_TREE, cv2.CHAIN_APPROX_SIMPLE)

    # for contour in contours:
    #     approx = cv2.approxPolyDP(contour, 0.01 * cv2.arcLength(contour, True), True)
    #     cv2.drawContours(image, [approx], 0, (0, 0, 0), 5)
    #     x = approx.ravel()[0]
    #     y = approx.ravel()[1]
    #     if len(approx) == 3: 
    #         cv2.putText(image, "Triangle", (x, y), cv2.FONT_HERSHEY_COMPLEX, 0.5, (0, 0 ,0))
        
    #     elif len(approx) == 4: 
    #         x ,y, w, h = cv2.boundingRect(approx)
    #         aspect_ratio = float(w/h)
    #         print(aspect_ratio)
    #         if aspect_ratio >= 0.95 and aspect_ratio <= 1.05:
    #             cv2.putText(image, "Square", (x, y), cv2.FONT_HERSHEY_COMPLEX, 0.5, (0, 0 ,0))
    #         else:
    #             cv2.putText(image, "Rectangle", (x, y), cv2.FONT_HERSHEY_COMPLEX, 0.5, (0, 0 ,0))
    #     elif len(approx) == 5:
    #         cv2.putText(image, "pentagon", (x, y), cv2.FONT_HERSHEY_COMPLEX, 0.5, (0, 0 ,0))
        
    #     elif len(approx) == 10:
    #         cv2.putText(image, "star", (x, y), cv2.FONT_HERSHEY_COMPLEX, 0.5, (0, 0 ,0))
        
    #     else: 
    #         cv2.putText(image, "circle", (x, y), cv2.FONT_HERSHEY_COMPLEX, 0.5, (0, 0 ,0))
            
    # return image

    # MAKES THE TEXT UNREADABLE FOR PYTESSERACT, CAN BE USED IN SOME CASES
    # image_eroded = cv2.erode(image_black, kernel, iterations=1)
    return image_black

def take_screenshot():
    screenshot_path = "KAHOOT_IMAGE.png"
    kahoot_screenshot = pyautogui.screenshot(region=(x, y, w, h)) # enter coordinates 
    kahoot_screenshot.save(screenshot_path)
    kahoot_screenshot_processed = pre_proccess_for_OCR(screenshot_path)
    cv2.imwrite("Processed.png", kahoot_screenshot_processed)
    return kahoot_screenshot_processed

take_screenshot()

def clean_TEXT(text):
    replacements = {
        '@': 'a',    
        '|': 'I',    
        '0': 'O',    
        '1': 'l',    
        '5': 'S',
        '8': 'B',
        '6': 'G',
        '9': 'g',
        '€': 'e',
        '§': 's',
        '!': 'i',
        '$': 'S',
        '+': 't',
        '(': 'c',
        ')': 'd',
        '<': 'c',
    }

    for wrong, right in replacements.items():
        text = text.replace(wrong, right)
    return text

def exctract_text_OCR(image):
    return pytesseract.image_to_string(image, config="--psm 6")


## API KEYS AND ETC FOR AI
openai.api_key = "your_api_key"


# CHECKING IF PRESSEED P, IF TRUE THEN TAKE SCREENSHOT SAVE IT PREPROCESS IT AND OCR IT AND THEN SHARE IT TO AI
while True:
    if keyboard.is_pressed("alt") and keyboard.is_pressed("p"):
        print("Taking image....")
        processed_image = take_screenshot()
        extracted_text = exctract_text_OCR(processed_image)
        clean_TEXT(extracted_text)
        print(extracted_text)
        print()
        print()

        ## AI STUFF        
        response = openai.chat.completions.create(
            model="gpt-4.1-mini", # personally use gpt-4.1-mini, but choose whatever model you would like!
            messages=[
                {"role": "system", "content": "You are an expert multiple-choice assistant—read the question and all answer choices carefully, solve accurately (including math and advanced topics like quantum physics), and respond only with the exact full text of the correct answer choice (no letters, no explanations, no extra words, and match spelling, punctuation, and capitalization exactly, if the text your are given does not include the right answer, then improvise and type the correct answer, sometimes, there might be True and False questions, also answer them as per i told."},
                {"role": "user", "content": extracted_text}
            ]
        )
        # PRINT THE RESPONSE OF AI TO THE OCR'ed TEXT
        print(response.choices[0].message.content)

        # BASICALLY DEBOUNCE 
        while keyboard.is_pressed("alt") and keyboard.is_pressed("p"):
            sleep(0.1)

    #ESCAPE METHOD
    if keyboard.is_pressed("esc"):
        print("Escaped") 
        break
```
