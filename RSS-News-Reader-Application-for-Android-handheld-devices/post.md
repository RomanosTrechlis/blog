
The application implements a simple RSS News feed reader. I started with a lot of auto-generated code and step by step I wrote my own code to fit the problem. A simple example is the implementation of list views. I started with auto-generated type and continue coding until I transformed it to one I liked.

The application has been tested on phone not on tablets, so I don’t know what really happens when executed on one ;-).

## 2. Use cases

1. Choose news feed and read it.
2. Manage the news feeds by choosing which to display.
3. Add more news feeds.
4. Setting the time interval between updates.


## 3. Design

The application consists of several activities:

1. The MainActivity draws an expandable list view of RSS feeds and the DetailsActivity draws the RSS items held by the selected RSS feed.
2. ManageActivity, NewsFeedActivity, SettingsActivity are simple activities, children of the MainActivity.
3. EditRssFeedActivity is a children of ManageActivity and enables the modification of RSS feed's information or their complete deletion.

The application contains two model classes:

1. RssFeed.java represents the information of an RSS feed.
2. RssItem.java represents the information of a single RSS feed item.

On the background an SQLite database is running with the help of DatabaseHanlder.java

Static content is Helper.java and the communication with the servers is facilitated with the RetrieveFeedTask.java
ManageCustomArrayAdapter and ExpCustomListAdapter are classes that customize the listview and the expandable list view in ManageActivity and MainActivity respectively.

A service (UpdateService.java) is implemented to facilitate the update schedule.


This was my introduction to the world of Android devices programming.
Download the RSS News Reader source code [here](https://github.com/RomanosTrechlis/RSSNewsReaderApp).