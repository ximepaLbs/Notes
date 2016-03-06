#title lvl1

##title lvl 2

Normal text
<!--comment-->
###alternatives 
quote : __Here is a quotation__
bold : **this is bold**
italic: *this is italic*
strikethrough : ~~this is strikethrough~~

The following content is blockquoted: 
>mail from toto:
>subject: minions need more bananas

Tables : (careful about spaces ) 
<!-- and : for alignment -->

 col1 | column 2 | col3
--- | ---:| ---| 
 bob | banana | toto


###lists
1. this
2. is
5. an ordereded list
3. where numbers don't matter
 	1. sublist 
 	2. sublist still with order

+ unordered list
+ continuation
	+ sublist
	+ (unordered too)

##Code
there is some inline code : `def toto(){toto=banana}`

and more (uninterpreted) in javascript: 
```javascript
<script type="text/javascript">document.location = 'https://example.com/?q=markdown+cheat+sheet';</script>
```

C Hello world :smile:
```c
#include <stdio.h>
int main(void){
	printf("Hello, world\n");
	return 0;
}
```

###links
[I'm a link to googole](https://www.google.com)
[I'm a link to google with a title ](https://www.google.com) "link to google"

[I am a numberred link inside a document][1]

that should be enough complete reference is [there](https://help.github.com/articles/github-flavored-markdown/)
[1] : http://9gag.com

###breaks
first line


another paragraph
another line in the same paragraph
---<!--line break -->

###Images
this is an image
![minion alternative text](http://www.hypeorlando.com/ride-for-veterans/wp-content/uploads/sites/27/2014/03/Chiquita-DM2-minion-banana-3.jpg)

or it can be written this way : ![minion alternative text 2][minion2]
[minion2]: http://minionslovebananas.com/images/gallery/preview/Chiquita-DM2-gallery_phil_eating_banana.jpg

At last, an image with a link : 
[![minion alternative text 3](http://wallpaper.pickywallpapers.com/samsung-corby/preview/despicable-me-2-banana.jpg)](http://www.google.com)

