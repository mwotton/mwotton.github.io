multitenancy in yesod
=====================

There are a few ways of doing multitenancy in yesod. Most of them
aren't very satisfactory.

To set the stage: in a multitenanted application, we expect to add new customers frequently, and each customer gets their own subdomain. This is why https://github.com/yesodweb/yesod/wiki/Domain-based-routing does not work for this use case: we don't statically know the subdomains we might need to respond to, and recompiling every time we get a new customer is not workable.

1. Just fetch the domain in the host header, strip off the root domain, done!

Well, almost. Let's say we have

    getCompanyJobR :: NumberedSlug -> Handler TypedContent

and we use that technique. We fetch
https://acmecorp.betterteam.com/explosives-manager-12 (it's a big
division) and see that we get the right result out. Bonus!

Except: when we create a typesafe link to a CompanyJobR, there is no place to
stash a company, and the link generated from that will be on the bare
root domain. None of our links in the XML feeds will work. Oops.

2. Write a middleware that maps acmecorp.betterteam.com/(whatever) to betterteam.com/acmecorp/(whatever)

this is a little better:

    getCompanyJobR :: CompanySubdomain -> NumberedSlug -> Handler TypedContent

has a space to stash a company subdomain. Unfortunately, while the
generated links will work (assuming the middleware passes requests
with no subdomain through), the generated links will be of the form
betterteam.com/acmecorp/explosives-manager-12

This will work ok, but it rather destroys the illusion we're working
hard to achieve that each company subdomain is a little island unto
itself. Nobody likes booking a penthouse and getting a hostel bunk.

3. Use the middleware technique, but override the link generation code
   somehow to produce the subdomain form

this could work, so long as we can stop config/routes generating link
code for that particular route.
