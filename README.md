import numpy as np
import cv2 , time
import asyncio
import io
import glob
import os
import sys
import time
import uuid
import requests
import cognitive_face as CF
import datetime
from tkinter import *
from urllib.parse import urlparse
from io import BytesIO
from PIL import Image, ImageDraw
from azure.cognitiveservices.vision.face import FaceClient
from msrest.authentication import CognitiveServicesCredentials
from azure.cognitiveservices.vision.face.models import TrainingStatusType, Person, SnapshotObjectType, OperationStatusType 
 
KEY = 'e8bf5adbb66c4d4b98fd2865250c3360'
ENDPOINT = 'https://facerec01.cognitiveservices.azure.com/'
face_client = FaceClient(ENDPOINT, CognitiveServicesCredentials(KEY))
PERSON_GROUP_ID = 'persn_prg05'
#face_client.person_group.create(person_group_id=PERSON_GROUP_ID, name='GROUP')
 
 
#اول جملة بتورجيك Persons جوا Group
#التانية بتعمل delete للكل
#جملة if بتعلم ديليت للشخص حسب الاسم انا هون شطبت رانيا
#for prs in face_client.person_group_person.list(PERSON_GROUP_ID):
    #print(prs.name,"  ",prs.person_id)
    #face_client.person_group_person.delete(PERSON_GROUP_ID , prs.person_id)
    #if(prs.name == "rania"):
        #face_client.person_group_person.delete(PERSON_GROUP_ID,prs.person_id)
        #print("rania deleted !")
 
def createPerson(personList,personNum,prs_images):
    for prs in personList:
        num=0
        personNum[num] = face_client.person_group_person.create(PERSON_GROUP_ID,name = prs)
        personNum[num].name = prs
        prs_images = []
        prs_images = [file for file in glob.glob('*.jpg') if file.startswith(prs)]
        for image in prs_images:
            w = open(image, 'r+b') # to read features of face
            face_client.person_group_person.add_face_from_stream(PERSON_GROUP_ID, personNum[num].person_id, w)      
    num+=1
 
 
def drawRec(image,faceFromCam,personName,currentDT ):
    img2 = Image.open(image)
    draw = ImageDraw.Draw(img2)
    draw.rectangle(getRectangle(faceFromCam), outline='red')
    draw.text(getRectanglelt(faceFromCam) , personName +' '+ currentDT.strftime("%c") , fill = 'black')
    img2.show()
 
def guiBox(personName ,currentDT):
    window = Tk()
    window.title("Welcome")
    window.geometry('250x200')
    lbl = Label(window, text=personName + ' '+ currentDT.strftime("%c"))
    lbl.grid(column=0, row=0)
    window.mainloop()
 
 
def getRectangle(faceDictionary):
    rect = faceDictionary.face_rectangle
    left = rect.left
    top = rect.top
    right = left + rect.width
    bottom = top + rect.height
    return ((left, top), (right, bottom))
 
def getRectanglelt(faceDictionary):
    rect = faceDictionary.face_rectangle
    left = rect.left
    top = rect.top
    right = left + rect.width
    bottom = top + rect.height   
    return (left, top)
 
def verificationWithRectAndGUI(faceList , mx_con ,prs_name ,groupID):
    nm = 0
    for faceID in faceList:
        mx_con = .55
        prs_name = 'NOT VERIFIED'
        
        for person in face_client.person_group_person.list(groupID):
            verf_face = face_client.face.verify_face_to_person(faceID , person.person_id , groupID)
            
            if(verf_face.confidence >mx_con):
                #found = 1
                mx_con = verf_face.confidence
                prs_name = person.name
                
 
        drawRec(cam_test , faces[nm] ,prs_name , currentDT)
        guiBox(prs_name , currentDT)
        nm+=1
        
 
def camCapture():
    camera = cv2.VideoCapture(0)
    cam_test = ''
    while True:
        return_value,image = camera.read()
        gray = cv2.cvtColor(image,cv2.COLOR_BGR2GRAY)
        cv2.imshow('image', gray)
        if cv2.waitKey(1)& 0xFF == ord('s'):
            cv2.imwrite('camCapt.jpg',image)
            cam_test = 'camCapt.jpg'     
            break
    camera.release()
    cv2.destroyAllWindows()
    return cam_test
 
createPerson(["khaled"],[0 for x in range(1000)],[])
 
 
cam_test = camCapture()
img = open(cam_test, 'r+b')
currentDT = datetime.datetime.now()
 
# Detect faces
face_ids = []
faces = face_client.face.detect_with_stream(img)
if not faces:
    sys.exit('No detected faces ! \nTry again')
    
else :
    for face in faces:
        face_ids.append(face.face_id)
 
for f in faces:
    print(f.face_id)        
mx_con = .55
prs_name = 'NOT VERIFIED'
 
verificationWithRectAndGUI(face_ids , mx_con ,prs_name ,PERSON_GROUP_ID)
'''
for f in faces:
    drawRec(cam_test,f ,prs_name , currentDT)
'''  
