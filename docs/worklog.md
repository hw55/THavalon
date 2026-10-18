the release is configured to run at render.com cloud (this server runs at thavalon.onrender.com). it is current;y configured to pick changes from the release branch and push. can you review and audit this idea? my wishlist -
1. creating a new release is not a automatic thing (to avoid new code breaking a running server)
2. a new release could easily be started from a web page by logged in owner (hopefully github.com)
3. a new release should create a new instance of the running server hosted at a separate url (that could be tested first)
4. after tests, the new release could be confirmed to run at the original url
i am using only the free version of render.com. how can i achieve all these needs?