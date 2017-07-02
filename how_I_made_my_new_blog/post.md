Today, I 'd like to write about building this blog. I could use a Haxo or Hugo, or any other platform that exists, however I ended up writing my own static site generator in [golang](https://golang.org/).

For my blog generator I used the core ideas of [Mario Zupan's](https://zupzup.org/) blog generator and altered it to fit my needs.

Zupzup explains clearly the concepts of a static blog generator, so go [here](https://zupzup.org/static-blog-generator-go/) and read his blog post and I'll continue from where he stopped.

## 1. Use a configuration file.

Even though his implementation is solid, changing variables in the code in order to produce a different blog and then compiling the project again wasn't efficient for me.

Additionaly, because I change blog posts on my computer and then push them to git I've implemented a local variation of datasource so I can use that local folder instead of a git repository. 

```go
    switch config.SiteInfo.DataSource.Type {
	case "git":
		ds := datasource.NewGitDataSource()
		_, err = ds.Fetch(config.SiteInfo.DataSource.Repository,
			config.SiteInfo.TempFolder)
	case "local":
		ds := datasource.NewLocalDataSource()
		_, err = ds.Fetch(config.SiteInfo.DataSource.Repository,
			config.SiteInfo.TempFolder)
	case "":
		log.Fatal("please provide a datasource in the configuration file")
	}
```

The configuration file looks like this:

```javascript
{
  "Author": "Romanos Trechlis",
  "BlogURL": "romanostrechlis.github.io",
  "BlogLanguage": "en-us",
  "BlogDescription": "Desc",
  "DateFormat": "2006-01-02 15:04:05",
  "ThemePath": "./static/",
  "BlogTitle": "RTB",
  "NumPostsFrontPage": 10,
  "DataSource": {
    "Type": "git",
    "Repository": "https://github.com/RomanosTrechlis/blog.git"
  },
  "TempFolder": "./tmp",
  "DestFolder": "./public"
}
```

The DataSource Type can also be *local* and the Repository can be a folder

```javascript
{
    "Type": "local",
    "Repository": "C:/Users/Romanos/Desktop/testLocal/blog/"
}
```

## 2. Use of cli to break functionality

Another issue I had was that I wish to download the blog content once and then generate the site multiple times with different templates. 

I also wish to see what my blog would look like before I push it. 

In order to achieve that I used flags to broke the functionality to distinct steps. So, if I want to generate, download and see the results I will run the command:

```bash
site-generator -fetch -generate -run
```

This will download my content, generate the site and run a local server.

## 3. Paging

Zupzup shows a fixed number of blog posts on the frontpage and uses archive to show all his blog posts.

For my blog I wished to show a fixed number of posts and give the visitor the ability to navigate to the next page with the next fixed number of blog posts.

I did that using the following code:

```go
    // frontpage
	paging := config.SiteInfo.NumPostsFrontPage
	numOfPages := getNumberOfPages(posts)
	for i := 0; i < numOfPages; i++ {
		to := destination
		if i != 0 {
			to = fmt.Sprintf("%s/%d", destination, i+1)
		}
		toP := (i + 1) * paging
		if (i + 1) == numOfPages {
			toP = len(posts)
		}
		generators = append(generators, &ListingGenerator{&ListingConfig{
			Posts:       posts[i*paging : toP],
			Template:    t,
			Destination: to,
			PageTitle:   "",
			PageNum:     i + 1,
			MaxPageNum:  numOfPages,
		}})
	}
```

That loop appends a new generator for every 10 post (the number of posts per page is given in the configuration file).

Additionally, in the html template:

```html
    <div id="paging">
        {{if ne .PageNum 0}}
        {{ if ne .PrevPageNum 0}}
        <a href="/{{if ne .PrevPageNum 1}}{{.PrevPageNum}}/{{end}}">
            <!-- Less Than icon by Icons8 -->
            <img style="vertical-align:middle" 
                 class="icon icons8-Less-Than"
                 width="30" height="30">
        </a>
        {{end}}
        <span><strong>{{.PageNum}}</strong></span>
        {{ if ne .NextPageNum 0 }}
        <a href="/{{.NextPageNum}}/">
            <!-- More Than icon by Icons8 -->
            <img style="vertical-align:middle" 
                 class="icon icons8-More-Than" 
                 width="30" height="30">
        </a>
        {{end}}
        {{end}}
    </div>
```

This code creates numbered folders that contain a new page with the next fixed amount of blog posts.

## 4. Share buttons

I also like share buttons to my blog posts. So, here is the template code:

```html
    {{if .IsPost}}
    <div id="share-buttons">
        <div class="share-button share-text">
            <span>Share</span>
        </div>
        <div class="share-button share-button-facebook" 
             data-share-url="{{.URL}}">
            <div class="box">
                <a href="https://www.facebook.com/sharer/sharer.php
                ?u={{.URL}}">
                    <!-- Facebook icon by Icons8 -->
                    <img class="icon icons8-Facebook" 
                         width="48" height="48" >
                </a>
            </div>
        </div>

        <div class="share-button share-button-twitter" 
             data-share-url="{{.URL}}">
            <div class="box">
                <a href="http://twitter.com/intent/tweet
                ?source=sharethiscom
                &text={{.PageTitle}}
                &url={{.URL}}&via=r_trechlis">
                    <!-- Twitter icon by Icons8 -->
                    <img class="icon icons8-Twitter" 
                         width="48" height="48" >
                </a>
            </div>
        </div>

        <div class="share-button share-button-gplus" 
             data-share-url="{{.URL}}">
            <div class="box">
                <a href="https://plus.google.com/share?url={{.URL}}">
                    <!-- Google Plus icon by Icons8 -->
                    <img class="icon icons8-Google-Plus" 
                         width="48" height="48" >
                </a>
            </div>
        </div>
    </div>
    {{end}}
```

IsPost is a boolean and URL the blog's url.

## 5. Future functionality: Upload

I also would like to automatically upload the generated blog to github.

For that end I made the **Endpoint** interface:

```go
type Endpoint interface {
	Upload(to string) error
}
```

This inteface must implement the Upload function.
