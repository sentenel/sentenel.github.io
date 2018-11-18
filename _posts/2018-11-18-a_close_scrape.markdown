---
layout: post
title:      "A Close Scrape"
date:       2018-11-18 21:38:33 +0000
permalink:  a_close_scrape
---

### A journey of gem creation, troubleshooting, and working through the problems with web scrapers.



**The Setup**

Recommended Reading is a CLI gem which allows a user to scrape popular sources for books to read and get reviews and information on those books from [Goodreads](https://www.goodreads.com/). After setting up the scaffold directory and going through the initial steps to create a gem with [Bundler](https://bundler.io/v1.16/guides/creating_gem.html), I decided on the [Amazon Bestsellers](https://www.amazon.com/best-sellers-books-Amazon/zgbs/books), [New York Times Bestsellers]https://www.nytimes.com/books/best-sellers/), [Barnes and Noble Top 100](https://www.barnesandnoble.com/), and [Publishers Weekly Top 10](https://www.publishersweekly.com/pw/nielsen/top100.html) as the list sources and got to work.

The basic library structure was straightforward: the CLI, a List class for book lists, a Book class for books, and two scrapers for getting lists from various sources and scraping book information from Goodreads. Goodreads allows for linking via ISBN, which made it easy to find the right books so long as that information was available on the list website. In the only case where it was not, the New York Times Bestsellers, going through their Amazon link reduced it to a solved problem.

For the UI the user would be presented with a choice of book lists to choose from, then a choice of books on that list, and finally options for specific information on a book such as a summary, rating statistics, reviews, or recommendations which could also be navigated to. The user could also return to the lists to view more material.



**Scraping Struggles**

More than once during the coding process, the scrapers needed to be modified when the target pages changed their structure or organization enough that it broke Nokogiri searches being used to find the relevant data. Additionally, when attempting to scrape reviews on Goodreads, I found them stored on separate pages, meaning scraping meant opening multiple pages in a row and causing a noticeable delay during use. Most troubling, though, was when seemingly out of nowhere, OpenURI began giving errors when trying to connect to Amazon. At first the problem was intermittent, but soon I was entirely unable to connect to Amazon to scrape it. Some research into the problem revealed that not only was this likely intentionally caused on their end, but scraping Amazon violates their ToS, and Amazon needed to be removed from the gem. Fortunately, the New York Times Best Sellers also linked to Barnes and Noble, and so it was only a little more work to get the ISBNs through there instead.



**Takeaways**

In addition to stretching my prior knowledge of the language, over the course of creating this gem I picked up some extra tidbits. Some points I found interesting:

* In addition to Nokogiri's `#css` search, using `#xpath` can get table data which cannot be found with `#css`, and `#at` removes the need for a 0 index or `#first` when know you're looking for the first (or only) element of the NodeSet

* Certain links encode special characters in urls as hex characters. While I was unable to find a simple method to handle this, some regex magic did the trick:
	`new_link = redirect_and_hex_link[/http%.*/].gsub(/%[1-9A-F]{1,2}/){|hex| hex[1, 2].to_i(16).chr}`

* In Ruby 2.3, open-uri cannot handle http-https redirects. This has been fixed in 2.4, but when using an earlier version, special measures may be needed, such as rescuing redirected links and trying them with an additional 's', or using a gem [created for the purpose](https://rubygems.org/gems/open_uri_redirections).

My biggest takeaway, though, is on scraping. Scraping is a very useful tool for quickly grabbing site data in an automated fashion, but it has a couple of major drawbacks. Any significant change to a site will likely break the scraper's searches, making it an unreliable method for long term use on any non-static site. Scraping also may violate terms of service, restricted for reasons ranging from business models to site load. For a site that is okay with their use and does not change often, scrapers are adequate for getting information comparatively quickly and easily, but in most other cases, APIs are the pattern when available.

