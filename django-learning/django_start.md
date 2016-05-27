# Django Start
### Django Project
* 创建一个django项目  
	
```
	django-admin startproject djangoweb
```
该命令会生成一个名为djangoweb的项目，项目包含一个名为djangoweb的package和manage.py文件。如下所示，

```
	djangoweb/
		djangoweb/
			__init__.py
			settings.py			#django项目的配置文件
			urls.py 			#文件指定了url和view的对应关系
			wsgi.py
		managy.py      			#命令行交互工具
```
* 启动  
	
在项目的根目录下执行如下命令,启动项目

```
	python manage.py runserver
```
还可以使用`python manage.py runserver port` 来制定端口，若不指定，则使用默认端口8000。

###Django app
* 创建一个django APP

```
	python manage.py startapp djangoapp 
```
该命令会生成一个名为djangoapp的package。如下所示，

```
	djangoapp/
		migrations/
		__init__.py
		admin.py
		apps.py
		models.py
		tests.py
		urls.py
		views.py
```
* Django project和Django app的对比  
` Django app是具有某种功能的web应用。Djando project可以理解为某个网站的配置和web应用的集合。`  

	> 	一个Django project可以包含多个Django app  
	>	一个Django app可以属于多个Django project

