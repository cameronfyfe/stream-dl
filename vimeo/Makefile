N_SEGMENTS := 1248 # may need to be checked/adjusted for different videos
SEGMENT_DELAY ?= 0 # can be used to emulate timing of stream requests the player would make

video-1080p.mp4:
	$(MAKE) _video RES=1080p OUT=$@

video-720p.mp4:
	$(MAKE) _video RES=720p OUT=$@

# Get this by using the web client to watch the video and copying it from the http requests for the stream.
# exp=X is the expiration time for the generated hmac token
hmacToken := <ENTER>
# Get this by looking at the next path parameter after the hmac token
groupId := <ENTER>
# Get this by looking at the url path for the audio .mp4 requests
audioId := <ENTER>
# Get these by changing the stream quality and checking the changed id in the stream requests for the .ms4 segments
ifeq ($(RES), 720p)
	videoId := <ENTER>
endif
ifeq ($(RES), 1080p)
	videoId := <ENTER>
endif
# Get this by looking at the url paths requested from the player
baseUrl := <ENTER>

audioUrl := $(baseUrl)/$(hmacToken)/$(groupId)/parcel/audio/$(audioId).mp4?r=dXM%3D
videoUrl = $(baseUrl)/$(hmacToken)/$(groupId)/sep/video/$(videoId)/chop/segment-$*.m4s?r=dXM%3D

segments := $(shell \
	for i in $$(seq 0 $(N_SEGMENTS)); do \
		echo segments/$(RES)/segment-$$i.m4s; \
	done \
)

audio-only.mp4:
	curl -L -o $@ $(audioUrl)
	
segments/$(RES)/segment-%.m4s:
	mkdir -p $(dir $@)
	curl -L -o $@ $(videoUrl)
	sleep $(SEGMENT_DELAY)

video-only-$(RES).ms4: $(segments)
	rm -f $@
	for seg in $(segments); do \
		cat $$seg >> $@; \
	done

video-only-$(RES).mp4: video-only-$(RES).ms4
	ffmpeg -i $< -c copy $@

.PHONY: _video
_video: video-only-$(RES).mp4 audio-only.mp4
	echo $(RES)
	ffmpeg \
		-i $< \
		-i $(word 2,$^) \
		-c:v copy \
		-c:a copy \
		$(OUT)
