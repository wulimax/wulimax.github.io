## 爬取gitboot网站数据 保存成html

```python
# -*- coding' : 'UTF-8 -*-


from bs4 import BeautifulSoup
import os
import requests
import FileUtlis


# 爬取 gitbook 数据


class PagesHtml:

	def getRequest(self,url):
		headers = {'cookie': self.getcookie()}
		resmpm = requests.get(url,headers=headers)
    
    # 获取cookie信息
	def getcookie(self):
		return ""

	def getFileObj(self,urlpath):
		headers = {'cookie': self.getcookie()}
		resmpm = requests.get(urlpath,headers=headers)
		soup=BeautifulSoup(resmpm.content,'lxml',from_encoding='utf-8') #具有容错功能
		#imag
		print("=========================")
		for a in soup.findAll('img',src=True):

			print(a['src'])
			if a['src'] == '..':
				continue
			if a['src'] == '../':
				continue
			str = a['src']
			self.echofile(self.hosturl + str[str.find('/')+1:])
		#js
		for a in soup.findAll('script',src=True):
			if a['src'] == '..':
				continue
			if a['src'] == '../':
				continue
			str = a['src']
			self.echofile(self.hosturl + str[str.find('/')+1:])
		#css
		for a in soup.findAll('link',href=True):
			if self.check(a['href'],'/') == True and self.check(a['href'],'#') == True :
				continue
			if a['href'] == '..':
				continue
			if a['href'] == '../':
				continue
			str = a['href']
			self.echofile(self.hosturl + str[str.find('/')+1:])
		#a
		for a in soup.findAll('a', href=True):
			if self.check(a['href'],'/') == True and self.check(a['href'],'#') == True :
				continue
			if self.check(a['href'],'https') == False:
				continue
			if a['href'] == '..':
				continue
			if a['href'] == '../':
				continue
			if self.check(a['href'],'#') == False:
				str = a['href']
				self.echofile(self.hosturl + str[str.find('#')+1:])
			else:
				str = a['href']
				self.echofile(self.hosturl + str[str.find('/')+1:])

	def check(self,string, sub_str):
		if sub_str in string:
			return False
		else:
			return True
	def echofile(self,url):
		path = ''
		if self.check(url,'#') == False:
			path = url.replace(self.hosturl+'#','')
		else:
			path = url.replace(self.hosturl,'')

		# print(path[:path.rfind("/")])
		path = path[:path.rfind("/")]
		path = path.replace('/','\\')
		# path = 'E:\\logs\\test\\'+path
		path = self.filepath +"\\"+ path
		if not os.path.exists(path):
			print('创建目录: '+path)
			os.makedirs(path)
		filename = url[url.rfind("/"):]
		filename = path + filename.replace('/','\\')
		if self.check(url,'ioa.tencent.com') == False:
			return
		if self.check(url,'?') == False:
			filename = filename[:filename.find("?")]
		if len(filename) < 10:
			return
		#判断是否在全局变量中
		if filename in self.dict:
			return
		else:
			self.dict[filename]= True
	   #判断文件是否存在
		if os.path.exists(filename):
			return

		print(filename)
		print(url)
		headers = {'cookie': self.getcookie()}
		resmpm = requests.get(url,headers=headers)
		with open(filename, 'ab') as file:
			file.write(resmpm.content)
			file.flush()
		self.getFileObj(url)
		# if self.check(url,'html') != False:
		# 	self.getFileObj(url)
		# 	return


if __name__ == "__main__":


	pagobj = PagesHtml()
	#爬取网址路径为后面拼接使用
    pagobj.hosturl = 'https://www.xxx.com/'
    # 本地存储路径(路径可能存在多级的情况建议多级目录)
	pagobj.filepath = 'E:\\iwiki'
	pagobj.status = True
	pagobj.dict = {}
    #开始爬取页面
	pagobj.getFileObj('https://www.xxx.com/index.html')



```