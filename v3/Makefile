default: glslx release

clean:
	rm -f src/core/shaders.sk www/main.js

debug: | node_modules
	node_modules/.bin/skewc src/*/*.sk --output-file=www/main.js

release: | node_modules
	node_modules/.bin/skewc src/*/*.sk --output-file=www/main.js --release

glslx: | node_modules
	node_modules/.bin/glslx shaders/all.glslx --output=src/core/shaders.sk --format=skew

watch-debug:
	node_modules/.bin/watch src 'clear && make debug'

watch-release:
	node_modules/.bin/watch src 'clear && make release'

watch-glslx:
	node_modules/.bin/watch shaders 'clear && make glslx'

node_modules:
	npm install
