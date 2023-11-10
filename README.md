# far-oeuf.com

#### Testing

To test locally:

``
$ cd docs
$ JEKYLL_ENV=development bundle exec jekyll serve
Configuration file: /Users/dougb/dev/thisdougb.github.io/docs/_config.yml
To use retry middleware with Faraday v2.0+, install `faraday-retry` gem
            Source: /Users/dougb/dev/thisdougb.github.io/docs
       Destination: /Users/dougb/dev/thisdougb.github.io/docs/_site
 Incremental build: disabled. Enable with --incremental
      Generating... 
      Remote Theme: Using theme jekyll/minima
       Jekyll Feed: Generating feed for posts
     Build Warning: Layout 'default' requested in 404.html does not exist.
                    done in 0.775 seconds.
 Auto-regeneration: enabled for '/Users/dougb/dev/thisdougb.github.io/docs'
    Server address: http://127.0.0.1:4000/
  Server running... press ctrl-c to stop.
```

Setting the ENV to development prevents GA4 activating.
