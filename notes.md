File storage
============

	derpibooru/
		images/
			[hash].[extension]
			0a0943b3fe931d507b65d8806f53d3f1.png
		json/
	e621/
		images/
			[hash].[extension]
			0a0943b3fe931d507b65d8806f53d3f1.png

The `[hash].[extension]` format suggested here is what e621 already uses for images you download, so that might be convenient here. It would also make it easy both to open and view images since they have appropriate extensions and to look up images from the json data we have since both e621 and derpibooru provide hashes in said json data. They do however both use different hashing algorithms (derpibooru: sha512, e621: md5), and there's always a chance that they change algorithm in the future which would be very problematic. **TODO:** Decide on what name stored images/media should have.

**TODO:** Do we want to store json entries (that is, individual posts/images/updates/tags..) as individual files, or just store each page we download as individual files?

Endpoints
=========

Format for each endpint:

1. a link to the endpoint
2. a list of parameters to the endpoint
3. a example of returned data
4. random notes

**TODO:** Decide on which endpoints we do care about

Derpibooru
----------

[images](https://derpibooru.org/images.json)

* constraint=
	- can be "updated", "id" or "created"
	- use "updated" when retrieving all posts
* order=
	- can be "a" or "d" for ascending or descending
	- use "a" when retrieving all posts (so that we start from the beginning and move forward in time)
* gte=
	- constratint field is greater than what this is set to
	- a time like "2017-07-18T01:49:59.798Z"
	- using the date command (using the beginning of unix epoch time): date -d '@0' -u +"%FT%T.%NZ"
* deleted=
	- can be set to 1 to include deleted posts
	- only "id", "created_at", "updated_at", and either "deletion_reason" or "duplicate_of" feilds are available for deleted posts
	- we probably want this set
* page=
	- page offset
	- it'll be better for us to use "gte" than this since as posts are updated they jump around in the list, and so a page doesn't necessarily always start at the same offset in the list which might lead to us missing entries

**Example:**

	{
	 "images": [
	  {
	   "id": "1488782",
	   "created_at": "2017-07-18T02:10:02.184Z",
	   "updated_at": "2017-07-18T02:12:10.1477Z",
	   "duplicate_reports": [],
	   "first_seen_at": "2017-07-18T02:10:02.184Z",
	   "uploader_id": "370728",
	   "score": 2,
	   "comment_count": 0,
	   "width": 2048,
	   "height": 2048,
	   "file_name": "image.png",
	   "description": "",
	   "uploader": "Football_Brony",
	   "image": "\/\/derpicdn.net\/img\/view\/2017\/7\/18\/1488782__safe_artist-colon-vanillashineart_oc_oc+only_oc-colon-silvershield_bowtie_clothes_glasses_laying+down_looking+away_solo_sweater.png",
	   "upvotes": 2,
	   "downvotes": 0,
	   "faves": 1,
	   "tags": "artist:vanillashineart, bowtie, clothes, glasses, laying down, looking away, oc, oc only, oc:silvershield, safe, solo, sweater",
	   "tag_ids": [
	    "42350",
	    "311706",
	    "240188",
	    "36457",
	    "81044",
	    "43771",
	    "21520",
	    "32747",
	    "28772",
	    "75957",
	    "40482",
	    "23269"
	   ],
	   "aspect_ratio": 1.0,
	   "original_format": "png",
	   "mime_type": "image\/png",
	   "sha512_hash": "c84bc1eaf1f06f660ddbe6b541f6755bb7b406568e3beff563c42a40251c05835800f94d1686d25ebcb777f47413eea60b563dbc5ce15353b81a6b584a34b101",
	   "orig_sha512_hash": "c84bc1eaf1f06f660ddbe6b541f6755bb7b406568e3beff563c42a40251c05835800f94d1686d25ebcb777f47413eea60b563dbc5ce15353b81a6b584a34b101",
	   "source_url": "",
	   "representations": {
	    "thumb_tiny": "\/\/derpicdn.net\/img\/2017\/7\/18\/1488782\/thumb_tiny.png",
	    "thumb_small": "\/\/derpicdn.net\/img\/2017\/7\/18\/1488782\/thumb_small.png",
	    "thumb": "\/\/derpicdn.net\/img\/2017\/7\/18\/1488782\/thumb.png",
	    "small": "\/\/derpicdn.net\/img\/2017\/7\/18\/1488782\/small.png",
	    "medium": "\/\/derpicdn.net\/img\/2017\/7\/18\/1488782\/medium.png",
	    "large": "\/\/derpicdn.net\/img\/2017\/7\/18\/1488782\/large.png",
	    "tall": "\/\/derpicdn.net\/img\/2017\/7\/18\/1488782\/tall.png",
	    "full": "\/\/derpicdn.net\/img\/view\/2017\/7\/18\/1488782__safe_artist-colon-vanillashineart_oc_oc+only_oc-colon-silvershield_bowtie_clothes_glasses_laying+down_looking+away_solo_sweater.png"
	   },
	   "is_rendered": true,
	   "is_optimized": true
	  },
	 ],
	 "interactions": []
	}

The last entry in the list of images is the most recent one with "order=a".

Derpibooru returns at most 15 images each time we download.

The "updated_at" variable in this data can be used for paging. Each time we download a list of images like this, take the "updated_at" field from the last (read: most recent) entry in the list and use that in the "gte=" parameter for the next download. A slight problem with this though is that the "updated_at" field is *rounded down*, and the "gte=" parameter compares to a more accurate time, so the comparison will always return *at least 1 duplicate*. We can either just ignore this and have at least 1 extra duplicate for every 15 entries, the only penalty being it takes a bit more space (theoretically about 7% more space before compression, though should be a lot less compressed since compression should deal with duplicates really well), or when we download a new set of entries we could look at the previous one and check if we have any duplicates and remove them. That would however complicate our code a bit, hence **TODO:** Figure out if we should deal with duplicates that we download or not?

**TODO:** Figure out what the "interactions" array is for

**TODO:** What's the difference between "sha512_hash" and "orig_sha512_hash"?

[tags](https://derpibooru.org/tags.json)

**TODO:** see what parameters are available

**Example:**

	[
	 {
	  "id": 42351,
	  "name": "solo female",
	  "slug": "solo+female",
	  "description": "A single female presented in a provocative or sexual manner. This tag does not apply to 'safe' rating images, please just use 'solo' for those.\r\n\r\nExample\r\n>>467158t",
	  "short_description": "A single female presented in a provocative or sexual manner",
	  "images": 101674,
	  "spoiler_image_uri": null,
	  "aliased_to": null,
	  "aliased_to_id": null,
	  "namespace": null,
	  "name_in_namespace": "solo female",
	  "implied_tags": "female, solo",
	  "implied_tag_ids": [
	   27141,
	   42350
	  ],
	  "category": null
	 }
	]

[comments](https://derpibooru.org/lists/recent_comments.json)

**TODO:** see what parameters are available

**Example:**

	[
	 {
	  "id": 6332668,
	  "body": "Luna looks so discouraged to be in the middle of that mess!",
	  "author": "RoygbivThePony",
	  "image_id": 1488459,
	  "posted_at": "2017-07-18T01:49:59.798Z",
	  "deleted": false
	 }
	]

e621
----

**TODO:** Research e621 endpoints

Downloader
==========

A general overview of how the downloader could work

Initialization
--------------

Main loop
---------
1. Download data

2. Decide on retry time

3. Find next download link

4. Download images/media, for each post do:
	1. Check if file is already downloaded

	2. Find link and download it

	3. Store as FILE_HASH.FILE_EXTENSION?
		# Should make the files viewable as well as easy to check for existance

	4. what do we do when downloads fail?
		# Deleted posts will of course not exist, so they should be skipped
		# For server errors/timeouts we should retry

5. Store json data

6. Compress json data when a certain amount has been downloaded

7. Store next download link
	# Do this last so we continue where we left of in case something fails/crashes/is interrupted

8. Sleep for the retry time we calculated before

