суббота, 26 сентября 2009 г. в 22:21:16

В продолжение статьи о [новинках в html5](https://kurapov.ee/rus/technology/web/html5_intro/), вот нашёл [демо](http://yehudab.com/apps/drag-n-drop/demo.html) где уже работает загрузка файлов обычным перетаскиванием в [браузер Firefox 3.6](http://ftp.mozilla.org/pub/mozilla.org/firefox/nightly/). Обычный drag-n-drop файлов пока в стабильных браузерах [максимально что могут](http://decafbad.com/blog/2009/07/15/html5-drag-and-drop) — создать события аналогичные перетаскиванию dom-элементов в определённые контейнеры.

Если взглянуть в код, то можно заметить что у события появляются такие параметры и методы:

`event.dataTransfer.files event.dataTransfer.mozTypesAt(0) event.dataTransfer.files[0].getAsBinary()`

А для отсылки файла используется обычный аяксный объект у которого появляется метод sendAsBinary(). Вобщем пока это выглядит как хак и видимо что надо больше внимания уделить спецификации html5, что-бы пользователь случайно не загрузил весь C:\Windows, что-бы можно было временно отложить загрузку или отфильтровать что конкретно надо загружать, или до какой глубины надо структуру подгружать и как обрабатывать.

### Jquery dnd-upload

Написал два jquery-плагина - один для поддержки Firefox 3.6 (Namoroka) HTML5 загрузки, другой для поддержки Google gears. Можно использовать оба вместе..

Самый простой вариант для gears:

`$('#drop_area').uploadg({gate:'http://somesite.com/upload_here_path/'});`

Заметьте что ключ получаемого у сервера файла будет "file" (в пхп соответсвенно $_FILES['file']). А вот расширенный пример кода инициализации..

`var dragndropConfig={ beforeLoad:function(){ this.gate=some_dynamic_file_upload_url+'?parentID='+SomeOtherDynamicObject.ID; if(SomeDisableReason){  this.can_proceed=false; } }, onProgress:function(event) { if (event.lengthComputable) { var percentage = Math.round((event.loaded * 100) / event.total); if (percentage &lt; 100) {  $('#some_progress_div').html('Uploading file..'+percentage+'%'); } } }, onComplete:function(event,txt) { $('#some_progress_div').html('done');  } }; if (window.google && google.gears){ $('#drop_area').uploadg(dragndropConfig); } else{ $('#drop_area').upload5(dragndropConfig); }`

Осторожно, Google gears всё ещё [имеют баги](http://groups.google.com/group/gears-users/browse_thread/thread/f6036bcc4a22cba5) связанные с такой загрузкой.


```javascript
/**
 * HTML5 drag and drop file jquery file uploading
 *
 * @author Artjom Kurapov <artkurapov@gmail.com>
 **/
$.upload5=function(input, opt){
	var ME=this;
	ME.option = {
		gate:'',
		can_proceed:true,
		
		dragEnterColor:'#ffc',
		dragExitColor:'#fff',
		
		beforeLoad:function() {},
		onProgress:function(event) {},
		onComplete:function(event) {}
	};

	if(typeof(opt)=='string'){
		opt={'gate':opt};
	}
	
	ME.option = $.extend(this.option,opt);

	 
	ME.upload = function (event) {
		
		event.stopPropagation();
		event.preventDefault();
        
		ME.option.beforeLoad();
		if(!ME.option.can_proceed){
			return false;
		}
		
		var data = event.dataTransfer;
		
        /*
        for (var i = 0; i < data.files.length; i++) {
            $('#image_list').prepend($('<img src="img/spinner.gif" width="16" height="16" />').css("padding", "33px"));
        }
        */
        
        var boundary = '------multipartformboundary' + (new Date).getTime();
        var dashdash = '--';
        var crlf     = '\r\n';

        /* Build RFC2388 string. */
        var builder = '';

        builder += dashdash;
        builder += boundary;
        builder += crlf;
        
        var xhr = new XMLHttpRequest()    

        
		xhr.upload.addEventListener("progress", ME.option.onProgress, false);
			
        xhr.open("POST", ME.option.gate, true);
        
        if($.browser.safari){
        	var formData = new FormData(); 
        	formData.append("file", data.files.item()); 
        	xhr.send(formData);
        }
        else{
        	
		
		    for (var i = 0; i < data.files.length; i++) {
		        var file = data.files[i];
		
		        /* Generate headers. */            
		        builder += 'Content-Disposition: form-data; name="file[]"';
		        if (file.fileName) {
		          builder += '; filename="' + file.fileName + '"';
		        }
		        builder += crlf;
		
		        builder += 'Content-Type: application/octet-stream';
		        builder += crlf;
		        builder += crlf; 
		
		        /* Append binary data. */
		        builder += file.getAsBinary();
		        builder += crlf;
		
		        /* Write boundary. */
		        builder += dashdash;
		        builder += boundary;
		        builder += crlf;
		    }
		
		    
		    /* Mark end of the request. */
		    builder += dashdash;
		    builder += boundary;
		    builder += dashdash;
		    builder += crlf;
		    
        	xhr.setRequestHeader('content-type', 'multipart/form-data; boundary=' + boundary);
	        
	        try{
	        	xhr.sendAsBinary(builder);        
	        }
	        catch(err){
	        	Note.set('error',t("Upload failed - use only latin-encoded filenames"),13);
	        	return false;
	        }
        }
        
        xhr.onload = function(event){
        	ME.option.onComplete(event,xhr.responseText);
        };
    }
			
	$(input).get(0).addEventListener('drop', ME.upload, false);
	
    $(input).get(0).addEventListener('dragenter',	function(event) { 
    	$(input).css("background-color", ME.option.dragEnterColor); 
    	event.stopPropagation();
    	event.preventDefault();
    }, false);
    
    $(input).get(0).addEventListener('dragover',	function(event) { 
    	$(input).css("background-color", ME.option.dragEnterColor); 
    	event.stopPropagation();
    	event.preventDefault();
    }, false);
    
    $(input).get(0).addEventListener('dragexit', 	function(event) { $(input).css("background-color", ME.option.dragExitColor); }, false);
      
	return ME;
}

$.fn.upload5 = function upload5(options){
	this.each(function() {
		var input = this;
		new jQuery.upload5(input, options);
	});
	
	return this;
};
```