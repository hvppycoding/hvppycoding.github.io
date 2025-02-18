---
title: "Captioning Datasets for Training Purposes"
excerpt: ""
date: 2025-02-19 01:00:00 +0900
classes: wide
header:
  overlay_image: /assets/images/unsplash-emile-perron.jpg
  overlay_filter: 0.5
  caption: "Photo by [**Emile Perron**](https://unsplash.com/@emilep) on [**Unsplash**](https://unsplash.com/)"
categories:
  - Stable Diffusion
---

This post is an archive of the [reddit's Captioning Datasets for Training Purposes](https://www.reddit.com/r/StableDiffusion/comments/118spz6/captioning_datasets_for_training_purposes/?utm_source=share&utm_medium=web3x&utm_name=web3xcss&utm_term=1&utm_content=share_button).
{: .notice--info}

In the spirit of how open the various SD communities are in sharing their models, processes, and everything else, I thought I would write something up based on my knowledge and experience so far in an area that I think doesn’t get enough attention: captioning datasets for training purposes.

## DISCLAIMER

I am not an expert in this field, this is simply a mix of what has worked for me, what I've learned from others, my understanding of the underlying processes, and the knowledge I've gained from working with other types of datasets.

I have found this workflow to be reasonably efficient when manually captioning a dataset considering the resulting quality of the captions compared to automated captioning. But be warned, if you are looking to caption hundreds of photos, it's still gonna take some time. To be clear, that means I am saying this method is not good for captioning truly large datasets with tens of thousands of images. Unless you are a masochist.

Sometimes I say "tag" and sometimes "caption". I was going to go through and fix it all, but I had captioning to do, so maybe I will make it uniform later.

I do not consider this document "finished". There is so much to learn, and the AI space is moving so fast, that it will likely never be finished. However, I will try to expand and alter this document as necessary.

My experience has primarily been with LoRA training, but some of the aspects here are applicable to all types of training.

## WHO IS THIS DOCUMENT FOR

I hope this document can be helpful to anyone who is somewhat seriously interested in training their own models in Stable Diffusion using their own datasets. If your goal is to quickly teach your face to a model, there are much better guides available which will have you up and running in a flash. But if your goal is to go a bit deeper, explore training in more depth, perhaps you can add this document to your resources.

## DATASET

Obtaining a good dataset is talked about extensively elsewhere, so I've only included the most important parts:

- high quality input means high quality output
- more quantity and more variety is better
- If you are forced to choose between quality and quantity, quality always wins.
- Upscale as a last resort, avoid it if possible. When I am forced to upscale, I use LDSR via Automatic1111.

## PREPARATION

Depending on how and what you are training, you may need to crop the photos to a specific width and height. Other types of training will bucket images into various sizes and do not require cropping. Look into what is required for the method of training you are doing, the model you are training on, and the program you are using to train your model with.

## CAPTIONING – GENERAL NOTES

The following recommendations are based on my experiments, my background work with other datasets, reading subject-matter papers, and borrowing from other successful approaches.

### Avoid automated captioning, for now

- BLIP and deepbooru are exciting, but I think it is a bit early for them yet.
- I often find mistakes and extremely repetitive captions, which take awhile to clean up.
- They struggle with context and with relative importance.
- I think it is faster to manually caption, rather than fix mistakes that BLIP/deepbooru made and still have to manually caption.

### Caption in the same manner you prompt

- Captioning and prompting are related.
- Recognize how you typically prompt. Verbose sentences? Short descriptions? Vague? Detailed?
- Caption in a similar style and verbosity as you tend to when prompting.

### Follow a set structure per concept

- Following a structure makes the process easier on you, and although I have no objective evidence, my intuition says that using a consistent structure to describe your dataset will benefit the learning process.
- You might have a structure you use for photographs and another structure you use for illustrations. But try to avoid mixing and matching structures when captioning a single dataset.
- have explained the structure I generally use below, which can be used as an example.

### Captions are like variables you can use in your prompts

- Everything you describe in a caption can be thought of as a variable that you can play with in your prompt. This has two implications:

1. You want to describe as much detail as you can about anything that isn’t the concept you are trying to implicitly teach. **In other words, describe everything that you want to become a variable**. `Example: If you are teaching a specific face but want to be able to change the hair color, you should describe the hair color in each image so that "hair color" becomes one of your variables.`
2. You don’t want to describe anything (beyond a class level description) that you want to be implicitly taught. **In other words, the thing you are trying to teach shouldn't become a variable**. `Example: If you are teaching a specific face, you should not describe that it has a big nose. You don’t want the nose size to be variable, because then it isn’t that specific face anymore.However, you can still caption "face" if you want to, which provides context to the model you are training. This does have some implications described in the following point.`

### Leveraging classes as tags

- There are two concepts here.

1. Using generic class tags will bias that entire class towards your training data. This may or may not be desired depending on what your goals are.
2. Using generic class tags provides context to the learning process. Conceptually, it is easier to learn what a "face" is when the model already has a reasonable approximation of "face".

- If you want to bias the entire class of your model towards your training images, use broad class tags rather than specific tags. `Example: If you want to teach your model that every man should look like Brad Pitt, your captions should contain the tag "man" but should not be more specific than that. This influences your model to produce a Brad Pitt looking man whenever you use the word "man" in your prompt. This also allows your model to draw on and leverage what it already knows about the concept of "man" while it is training.`
- If you want to reduce the impact of your training on the entire class, include specific tags and de-emphasize class tags. `Example: If you want to teach your model that only "ohwxman" should look like Brad Pitt, and you don't want every "man" to look like Brad Pitt you would not use "man" as a tag, only tagging it with "ohwxman". This reduces the impact of your training images on the tag "man", and strongly associates your training images with "ohwxman". Your model will draw on what it knows about "ohwxman", which is practically nothing \*see note\*, thus building up knowledge almost solely from your training images which creates a very strong association.`

\*NOTE\* This is simplified for the sake of understanding. This would actually be tokenized into two tokens, "ohwx" and "man", but these tokens would be strongly correlated for training purposes, which should still reduce the impact on the overall class of "man" when compared to training with "man" as a token in the caption. The math it all is quite complex and well beyond the scope here.
{: .notice--info}

### Consistent Captioning

- Use consistent captions across all of your training. This will help you better consistently invoke your concept when prompting. I use a program to aid me with this, ensuring that I always use the same captions.
- Using inconsistent tags across your dataset is going to make the concept you are trying to teach harder for SD to grasp as you are essentially forcing it to learn both the concept and the different phrasings for that concept. It’s much better to have it just learn the concept under a single term.
- `For example, you probably don't want to have both "legs raised in air" and "raised legs" if you are trying to teach one single concept of a person with their legs up in the air. You want to be able to consistently invoke this pose in your prompt, so choose one way to caption it.`

### Avoid Repetition

Try to avoid repetition wherever possible. Similar to prompting, repeating words increases the weighting of those words.

As an example, I often find myself repeating the word "background" too much. I might have three tags that say "background" (`Example: simple background, white background, lamp in background`). Even though I want the background to have low weight, I've unintentionally increased the weighting quite a bit. It would be better to combine these or reword them (`Example: simple white background with a lamp`).

### Take Note of Ordering

- Again, just like with prompting, order matters for relative weighting of tags.
- Having a specific structure/order that you generally use for captions can help you maintain relative weightings of tags between images in your dataset, which should be beneficial to the training process.
- Having a standardized ordering makes the whole captioning process faster as you become familiar with captioning in that structure.

### Use Your Models Existing Knowledge to Your Advantage

- Your model already produces decent results and reasonably understands what you are prompting. Take advantage of that by captioning with words that already work well in your prompts.
- You want to use descriptive words, but if you use words that are too obscure/niche, you likely can't leverage much of the existing knowledge. Example: you could say "sarcrastic" or you could say "mordacious". SD has some idea of what "sarcastic" conveys, but it likely has no clue what "mordacious" is.
- You can also look at this from the opposite perspective. If you were trying to teach the concept of "mordacious", you might have a dataset full of images that convey "sarcrastic" and caption them with both the tags "sarcastic" and "mordacious" side by side (so that they are close in relative weighting).

## CAPTIONING – STRUCTURE

I use this mainly for people / characters, so it might not be quite as applicable to something like fantasy landscapes, but perhaps it can give some inspiration.

I want to emphasize again that I am not saying this is the only or best way to caption. This is just how I have found success with my own captions on my own datasets. My goal is simply to share what I do and why, and you are free to take as much or little inspiration from it as you want.

### General format

- `<Globals> <Type/Perspective/"Of a..."> <Action Words> <Subject Descriptions> <Notable Details> <Background/Location> <Loose Associations>`

### Globals

- This is where I would stick a rare token (e.g. "ohwx") that I want heavily associated with the concept I am training, or anything that is both important to the training and uniform across the dataset Examples: man, woman, anime

### Type/Perspective/"of a..."

- Broad descriptions of the image to supply context. I usually do this in "layers".
- What is it? `Examples: photograph, illustration, drawing, portrait, render, anime.`
- Of a... `Examples: woman, man, mountain, trees, forest, fantasy scene, cityscape`
- What type of X is it (x = choice above)? `Examples: full body, close up, cowboy shot, cropped, filtered, black and white, landscape, 80s style`
- What perspective is X from? `Examples: from above, from below, from front, from behind, from side, forced perspective, tilt-shifted, depth of field`

### Action Words

- Descriptions of what the main subject is doing or what is happening to the main subject, or general verbs that are applicable to the concept in the image. Describe in as much detail as possible, with a combination of as many verbs as you want.
- The goal is to make all the actions, poses, and whatever else active that is happening into variables (as described in point 3 of "Captioning – General") so that, hopefully, SD is better able to learn the main concept in a general sense rather than only learning the main concept doing specific actions.
- `Using a person as an example: standing, sitting, leaning, arms above head, walking, running, jumping, one arm up, one leg out, elbows bent, posing, kneeling, stretching, arms in front, knee bent, lying down, looking away, looking up, looking at viewer`
- `Using a flower as an example: wilting, growing, blooming, decaying, blossoming`

### Subject Descriptions

- As much description about the subject as possible, without describing the main concept you are trying to teach. Once again, think of this as picking out everything that you want to be a variable in your prompt.
- `Using a person as an example: white hat, blue shirt, silver necklace, sunglasses, pink shoes, blonde hair, silver bracelet, green jacket, large backpack`
- `Using a flower as an example: pink petals, green leaves, tall, straight, thorny, round leaves`

### Notable Details

- I use this as a sort of catch-all for anything that I don’t think is quite "background" (or something that is background but I want to emphasize) but also isn’t the main subject.
- Normally the part of the caption going in this spot is unique to one or just a few training images.
- I predominately use short captions in Danbooru-style, but if I need to describe something more complex I put it here.
- `For example, in a photo at a beach I might put "yellow and blue striped umbrella partially open in foreground".`
- `For example, in a portrait I might put "he is holding a cellphone to his ear".`

### Background / Location

- Fairly self-explanatory. Be as descriptive as possible about what is happening in the images background. I tend to do this in a few "layers" as well, narrowing down to specifics, which helps when captioning several photos.
- `For example, for a beach photo I might put (separated by the three "layers"):`

1. Outdoors, beach, sand, water, shore, sunset
2. Small waves, ships out at sea, sandcastle, towels
3. the ships are red and white, the sandcastle has a moat around it, the towels are red with yellow stripes

### Loose Associations

- This is where I put any final loose associations I have with the image.
- This could be anything that pops up in my head, usually "feelings" that I feel when looking at the image or concepts I feel are portrayed, really anything goes here as long as it exists in the image.
- Keep in mind this is for loose associations. If the image is very obviously portraying some feeling, you may want it tagged closer to the start of the caption for higher weighting.
- `For example: happy, sad, joyous, hopeful, lonely, sombre`

## THE BOORU DATASET TAG MANAGER

You’ve got a dataset. You’ve decided on a structure. You’re ready to start captioning. Now it’s time for the magic part of the workflow: [BooruDatasetTagManager](https://github.com/starik222/BooruDatasetTagManager)(BDTM). This handy piece of software will do two extremely important things for us which greatly speeds up the workflow:

1. Tags are preloaded in \*\\tags\\list.tag, which can be edited. This gives us auto-complete for common tags, allows us to double-click common tags so we don’t need to type it out, etc.
2. It enables you to easily be consistent with your captioning by displaying already-used captions so that you can easily add it to an image without typing it out.

As an added bonus, it helps when you're forgetful. Sometimes I forget that standing with most of your weight on one foot (but with both feet on the ground) is called contrapposto. But I have it saved as a tag, and usually remember it starts as "contra". Thankfully auto-complete is there to save the day. Seriously, having all of these tags at your fingertips is a huge difference from trying to remember a bunch of tags or having booru sites open in other tabs.

## THE PROCESS

1. Place all of your images in a folder and then navigate there in the BDTM UI, selecting the folder with your images.
2. At the top, press "View" and then "Show preview" to see the selected image.
3. If you have any globally applicable tags, add them on the right side of the UI. You can select where these global tags appear (top, bottom, or at a specific position in the list).
4. Select your image on the left and begin adding tags, remembering to follow you structure as best as possible. As you type, the tags will show auto-complete options from the list.tag file which you can select, or you can type in your own custom ones.
5. Each tag you have used anywhere in that dataset will show on the right side (under "All tags"). You can double-click a tag from the "All tags" section to apply it to the currently selected image, saving tons of time and ensuring tag consistency across your dataset
6. Once all of your images are tagged, go back to the start and do it again. This time look at your tags and make sure they are ordered appropriately according to the weighting you want (you can drag them to reorder if necessary), make sure they follow your structure, check for missing tags, etc.

And that’s it. I patiently look at every image and add any tags I think are applicable, aiming to have at least one to two tags in each of the categories of my prompt structure. I usually have between 8 and 20 tags per image, though sometimes I might have even more.

Over time, I have edited the provided list.tag file removing many of the tags I’ll never use and adding a bunch of tags that I use frequently, making the whole process even easier.

## FULL EXAMPLE OF A SINGLE IMAGE

This is an example of how I would caption a single image I picked off of safebooru. *We will assume that I want to train the style of this image and associate it with the tag "ohwxStyle", and we will assume that I have many images in this style within my dataset.*

Sample Image: https://safebooru.org/index.php?page=post&s=view&id=3887414

- Globals: ohwxStyle
- Type/Perspective/Of a: anime, drawing, of a young woman, full body shot, from side
- Action words: sitting, looking at viewer, smiling, head tilt, holding a phone, eyes closed
- Subject description: short brown hair, pale pink dress with dark edges, stuffed animal in lap, brown slippers
- Notable details: sunlight through windows as lighting source
- Background/location: brown couch, red patterned fabric on couch, wooden floor, white water-stained paint on walls, refrigerator in background, coffee machine sitting on a countertop, table in front of couch, bananas and coffee pot on table, white board on wall, clock on wall, stuffed animal chicken on floor
- Loose associations: dreary environment

All together: ohwxStyle, anime, drawing, of a young woman, full body shot, from side, sitting, looking at viewer, smiling, head tilt, holding a phone, eyes closed, short brown hair, pale pink dress with dark edges, stuffed animal in lap, brown slippers, sunlight through windows as lighting source, brown couch, red patterned fabric on couch, wooden floor, white water-stained paint on walls, refrigerator in background, coffee machine sitting on a countertop, table in front of couch, bananas and coffee pot on table, white board on wall, clock on wall, stuffed animal chicken on floor, dreary environment

The best part is, I can set all of those "global" ones in BDTM to apply to all of my images. I've now also got all of those tags ready just a double-click away, so if my next image is also a full body shot, from the side, sitting... I just double-click it. Much easier than typing it out again.

## TRAIN

Time to start training! I don't have much to write here other than experiment. There is no golden number of steps or guaranteed results when it comes to training. That's why it's fun to experiment. And now you can experiment knowing that you have an extremely high quality dataset, allowing you to really hone-in on the appropriate training settings.

## MISC THOUGHTS AND REFERENCES

- I always try to remind myself that we are just gently guiding the learning process, not controlling it. Your captions help point the learning process in the right direction, but the captions are not absolute. Inferences will be made on things in the image that weren't captioned, associations will be made between tags and parts of the image you didn't intend, etc. Try to guide, but trust in the training and the quality of your images as well.
- Danbooru/safebooru tags are great. I mean, there's a lot of trash ones that hold no meaning, but take a look at the Danbooru wiki for tag group "Posture" as an example. Dozens of specific words for different arm positions, leg positions, etc. You might just find that one specific word you've been searching for that describes the style/pose/lighting/whatever by crawling through the danbooru tags and wiki. Maybe you've always wanted someone posing with that ballerina style foot where the toes are pointed downwards. Well it's called plantar flexion; thanks danbooru tags.