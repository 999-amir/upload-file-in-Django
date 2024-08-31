📤how to upload-file in django📤<br>
1- pip install pillow<br>
2- define static and media in settings
```python
# core/settings
# Static files (CSS, JavaScript, Images)
STATIC_URL = '/static/'
STATIC_ROOT = BASE_DIR / 'static/'
STATICFILES_DIRS = [BASE_DIR / 'media/']

# media files (img)
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media/'
```
3- add to base-url
```python
# core/urls
# media & static path
if settings.DEBUG:
    urlpatterns += static(settings.STATIC_URL, document_root=settings.STATIC_ROOT)
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```
4- create model
```python
class FileModel(model.Model):
    file = models.FileField(upload_to='blog-files/%y/%m/%d/')
    
    @property
    def filename(self):
        return os.path.basename(self.file.name)

    @property
    def is_image(self):
        if self.filename.lower().endswith(('.jpg', 'jpeg', '.png', '.gif', '.svg', '.webp')):
            return True
        else:
            return False
```
5- create form-field
```python
# app/forms.py
class UploadFileForm(forms.Form):
    file = forms.FileField()
```
6- create view
```python
# app/views.py
class UploadFileView(View):
    form_class = UploadFileView

    def get(self, request, blog_title):
        forms = self.form_class()
        context = {
            'forms': forms,
            'files': FileModel.objects.all()    # show previous saved files
        }
        return render(request, 'uploadfile.html', context)

    def post(self, request, blog_title):
        forms = self.form_class(request.POST)
        try:    # if file has been selected
            file = request.FILES['file']
        except MultiValueDictKeyError:      # if user want to post empty file-field
            messages.warning(request, 'field should be filled', 'amber-600')
            context = {
                'forms': forms,
                'files': FileModel.objects.all()
            }
            return render(request, 'show-files.html', context)
        FileModel.objects.create(file=file)
        messages.success(request, 'new content added', 'green-600')
        return redirect('file:file-uploader')
```
7- create html
```html
<!--show previous saved files-->
{% for file in files %}
    {% if file.is_image %}
        <img src="{{ file.url }}">
    {% else %}
        <a href="{{ file.url }}" download="">{{ file.filename }}</a>
    {% endif %}
{% endfor %}
<!--upload new file-->
<form method="post"  enctype="multipart/form-data">
    <input type="file" name="file">
    <button type="submit">submit</button>
</form>
```
