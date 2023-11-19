# Notion Global Tag Database Step-by-Step

**Comments are enabled on this page 👍** 

This page describes how to implement the suggestion described in the Reddit Post linked to the right. This creates a **"Global Tag Database"** that can be used to keep tags consistent across databases.

**Known Issue:** Please note that moving pages between databases will not move the tags. For example: If  I move "Dog Food" from Task Table to Shopping List, [Pets](Notion%20Global%20Tag%20Database%20Step-by-Step%207f6514f3f68f4ab889d206e618331557/%5BEx%5D%20TAGS%20DATABASE%2054fc518590e044f1bdaf8f5c40696f5a/Pets%20811860dc4a29406d901d139e84f79e48.md)  will not show up under Tags anymore. HOWEVER If you move the page BACK to Task Table it will show up again. The only workaround would be to add [Pets](Notion%20Global%20Tag%20Database%20Step-by-Step%207f6514f3f68f4ab889d206e618331557/%5BEx%5D%20TAGS%20DATABASE%2054fc518590e044f1bdaf8f5c40696f5a/Pets%20811860dc4a29406d901d139e84f79e48.md)  again once it's in Shopping List. 

So, if you intend to move pages around you will need to add tags once for each table the page will live in.

[r/NotionSo - Notioneers! Level up your tagging!](https://www.reddit.com/r/NotionSo/comments/ayvkcs/notioneers_level_up_your_tagging/)

Credit to the above redditor for this idea

- **Change log**
    
    3/22/2019 - after testing the above post for my own use, I added instructions and turned the page public
    
    11/2/2019 - this page got some activity! I added and rearranged a few things
    
    12/6/2019 - ^^^^
    
    2/14/2020❤ - Checking in. Updating some dates. Happy Valentine's Day
    
    6/2/2020 - Can't believe that was only four months ago.  Anyway, I made @Jereon Strompf's comment more visible with a callout. @Sascha Liem requests screenshots which I will hopefully get to one day! 
    
    11/30/2020 - So many comments this year! Thanks for all the kudos everyone.  I updated some dates so the calendar will show things all year. I resolved all the comments at the top of the page, but you can find them at the bottom since they contained some useful thoughts and info. 
    
    1/5/2021 - Happy New Year! 🎉 Answered a couple of comments. Hid page comments. (They are harder to Reply to) Also added a BONUS Step 4 and some Roll Up columns. Made note of a known issue. 
    
    1/20/2022 It’s been over a year. The new comment system makes it easier to reply! I wish I hadn’t resolved all those old ones... oh well. 
    
    6/6/2022 I’m still here!
    

<aside>
⚠️ NOTE: This *replaces* the default "Tags" property with a *relational* property

</aside>

### Part One - Create your Tag Database (Gallery View)

1. Create a New Page; Select Table; Name this "Tags Database". This is where all your tags will go. You don't need to add them all at once
2. In the table you just created, add a row for each Tag you want to use. You do not need to add them all at once. If you do not want to see the "folded page" icon you will need to open each tag page and Add an icon to it.
    
    <aside>
    📌 How to add an icon: Create the tag, open it, and then add an icon :) -@Jeroen Strompf
    
    </aside>
    
3. To create the view shown in the reddit post, create a Gallery View 
4. Under Properties>Card Preview, Select None.

### Part Two - Connecting Databases to your Tag Database

1. In a new or existing database, remove (or rename) the existing "Tags" Property - we won't be using that for this purpose
2. Add a Relational Property that links to the Tag Database created above

### Part Three - Using your Tag Database

1. To add tags, edit the Relational property created in step 2 
2. Click the + button next to the tag(s) you want to add, or the x next to one you want to remove, 
- See the screenshot below for a visual example:
    
    ![Notion%20Global%20Tag%20Database%20Step-by-Step%207f6514f3f68f4ab889d206e618331557/Untitled2.png](Notion%20Global%20Tag%20Database%20Step-by-Step%207f6514f3f68f4ab889d206e618331557/Untitled2.png)
    

### Part Four - Bonus

Here are some extra things you can do:

1. In your Tags Database itself, you should see a Relational Column for each Table you have been using the tags in. You can shorten these column names and make some useful views ([](Notion%20Global%20Tag%20Database%20Step-by-Step%207f6514f3f68f4ab889d206e618331557/%5BEx%5D%20TAGS%20DATABASE%2054fc518590e044f1bdaf8f5c40696f5a.md))
2. You can also make a Rollup Column for each of these and configure them to Count All to show how many times each tag is used in each table. Add a Formula column that adds all of these together and now you have a column that counts how many times each tag is used. ([](Notion%20Global%20Tag%20Database%20Step-by-Step%207f6514f3f68f4ab889d206e618331557/%5BEx%5D%20TAGS%20DATABASE%2054fc518590e044f1bdaf8f5c40696f5a.md))
3. Open a Tag page (like "[🐾Pets](Notion%20Global%20Tag%20Database%20Step-by-Step%207f6514f3f68f4ab889d206e618331557/%5BEx%5D%20TAGS%20DATABASE%2054fc518590e044f1bdaf8f5c40696f5a/Pets%20811860dc4a29406d901d139e84f79e48.md)") and customize the look by dragging properties around and clicking the three dots in the top right, choosing "Customize Page", then setting properties to "Always Hide" or "Hide when empty"

## Examples

---

[[Ex] TAGS DATABASE](Notion%20Global%20Tag%20Database%20Step-by-Step%207f6514f3f68f4ab889d206e618331557/%5BEx%5D%20TAGS%20DATABASE%2054fc518590e044f1bdaf8f5c40696f5a.csv)

[[Ex] Tagged Project Database](Notion%20Global%20Tag%20Database%20Step-by-Step%207f6514f3f68f4ab889d206e618331557/%5BEx%5D%20Tagged%20Project%20Database%20d680b9ba91d245f6bea815c3252a29d1.csv)

[[EX] Tagged List](Notion%20Global%20Tag%20Database%20Step-by-Step%207f6514f3f68f4ab889d206e618331557/%5BEX%5D%20Tagged%20List%20aa4ce1b6eb994ed1baebb5f697cbe5fc.csv)

[[Ex] Tagged Task Table](Notion%20Global%20Tag%20Database%20Step-by-Step%207f6514f3f68f4ab889d206e618331557/%5BEx%5D%20Tagged%20Task%20Table%2054e50ac074354def9d5483be58b9c3a9.csv)

[[Ex] Tagged Shopping List](Notion%20Global%20Tag%20Database%20Step-by-Step%207f6514f3f68f4ab889d206e618331557/%5BEx%5D%20Tagged%20Shopping%20List%20d5a0acb0db934f68a32128e088d63dc3.csv)

[[Ex] Tagged Calendar](Notion%20Global%20Tag%20Database%20Step-by-Step%207f6514f3f68f4ab889d206e618331557/%5BEx%5D%20Tagged%20Calendar%203ec7be62ed0e44aca988ad6298e4b8f6.csv)

[Comments ](Notion%20Global%20Tag%20Database%20Step-by-Step%207f6514f3f68f4ab889d206e618331557/Comments%20e9e30858aa5a482b939ad5d6356b82b8.md)