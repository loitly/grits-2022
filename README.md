## grits-2022

This slide is created using [reveal-md](https://github.com/webpro/reveal-md) 
with added custom compose.yml file.


### Quick-start

At terminal, run

    docker compose up dev

Then, go to http://localhost:8000/slides.md  
Start editing slides.md to see it renders in the browser.  


### Create static pages

To generate static pages, so it can be hosted on GitHub or any web server.  

At terminal, run

    docker compose run static

This will export the slide into a stand-alone HTML pages including scripts and 
stylesheets under `docs` directory.  You can then use GitHub pages features to host it.

### Print PDF version

To view slide as a PDF printable page, 

At terminal, run

    docker compose up dev

Then, go to http://localhost:8000/slides.md?print-pdf
