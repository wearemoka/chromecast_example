#!/bin/bash

up:
	docker-compose up

upd:
	docker-compose up -d

clean:
	docker-compose exec cordovapp cordova clean

sh:
	docker-compose exec cordovapp sh

build-dev:
	docker-compose exec cordovapp npm run build
	docker-compose exec cordovapp cordova build android
	cp platforms/android/app/build/outputs/apk/debug/app-debug.apk apk/
