### ОПИСАНИЕ:
данный проект запускает на своём сервере программу ffmpeg которая подключается через консольную команду
к трансляции любой IP камеры по RTSP протоколу и начинает декодировать этот поток в формат поддерживаемый браузерами MPEG
путём создания каждые 10 секунд маленьких видео файликов, которые формируют LIVE трансляцию при обращении к главному файлу index.m3u8
через тэг video в HTML разметке
### ИНСТРУКЦИЯ:
1) подключаем камеру к роутеру, делаем базовую настройку, устанавливаем пароль
2) заказываем статический IP у провайдера
3) пробрасываем через web интерфейс роутера 554 порт (TCP/UDP) либо вместе (если есть такая опция) либо по отдельности TCP 554 и UDP 554
4) наша камера доступна по RTSP протоколу такого вида:  rtsp://{login}:{password}@{ip роутера}
5) для проверки работоспособности можно использовать VLC плеер с функцией подключения по URL и вставляем туда нашу rtsp ссылку, если всё сделано правильно то получаем LIVE трансляцию доступную в интернете
6) Далее приступаем к интеграции камеры на сайт: поднимаем сервер, накатываем LAMP вручную или ISP manager
7) Качаем и устанавливаем FFmpeg (очень мощная и многофункциональная программа, конкретно нам нужна для конвертации rtsp потока в mp4) чтобы она вызывалась из консоли (если Linux или Mac система - то устанавливаем пакет, если Windows - то качаем архив с официального сайта, распаковываем и указываем переменную окружения). [скачать FFMPEG](https://ffmpeg.org/download.html)
8) Выполняем эту команду: ffmpeg -i rtsp://{login}:{password}@{ip роутера} -c:v libx264 -c:a aac -f hls -hls_time 10 -hls_list_size 6 -hls_list_size 10 -start_number 1 /{путь к папке где будут создаваться mp4 файлы}/index.m3u8
После чего консоль начинает принимать пакеты с потока и создавать мини файлы трансляции в указанную нами папку
9) И завершающий этап - создаём HTML страницу с тэгом VIDEO и подключаем JS библиотеку для обработки HLS формата в который вставляем ссылку на наш файл index.m3u8
```html
<video controls id="video"></video>
<script src="https://cdn.jsdelivr.net/npm/hls.js@1"></script>
<script>
	let video = document.getElementById('video');
	let videoSrc = '/hls/index.m3u8';
	if (video.canPlayType('application/vnd.apple.mpegurl')) {
		video.src = videoSrc;
	} else if (Hls.isSupported()) {
		var hls = new Hls();
		hls.loadSource(videoSrc);
		hls.attachMedia(video);
	}
</script>
```

### РЕКОМЕНДАЦИИ:
* Использовать утилиту supervisor для контроля бесперебойной работы команды ffmpeg
* сделать CRON команду на систематическую очистку mp4 файлов которые будут создаваться каждый 10 секунд в папке которую мы указали