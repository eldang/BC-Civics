# MLA finder

Tools to make it easier for people to identify the MLA who's actually supposed to listen to them.

## For individuals

If you are just looking for your own MLA, there's no need to use anything here because existing services have you covered.

If you know your full 6-character postcode, use the search box at https://www.leg.bc.ca/members

If you know the rest of your address but not the postcode, then:

1. Use https://elections.bc.ca/resources/maps/find-your-district/ to find your district
2. Look up your district at https://www.leg.bc.ca/members/find-mla-by-constituency to look up your district, and it will show you your MLA's name and contact info.

## For bulk lookups

What I'm actually interested in here is building tools for bulk lookups so that when organisers have a list of people interested in helping a campaign, they can direct those people to the appropriate MLA, and/or build groups of people to approach their MLA together.

I haven't built anything yet, so far I'm just using this repository to keep track of data sources and potentially useful existing tools.

### Full postcodes

Unfortunately the boundaries of full postcodes are proprietary data and expensive to buy.  But the API behind https://www.leg.bc.ca/members isn't terribly difficult to use:

`POST` request to `https://lims.leg.bc.ca/graphql`

Request body: `{"operationName":"GetMLAName","variables":{"postalCode":"X0X0X0"},"query":"query GetMLAName($postalCode: String!) { allPostalCodeLookups(condition: {postalCode: $postalCode}) { nodes { constituencyByConstituencyId { memberByMemberId { firstName lastName __typename } } } }}"}`

Gets a response like this:

```json
{
  "data": {
    "allPostalCodeLookups": {
      "nodes": [
        {
          "constituencyByConstituencyId": {
            "memberByMemberId": {
              "firstName": "Darlene",
              "lastName": "Rotchford",
              "__typename": "Member"
            }
          }
        }
      ]
    }
  }
}
```

And then the individual MLA pages seem to reliably have URLs in the form `https://www.leg.bc.ca/members/43rd-Parliament/lastName-firstName` so the URL is easy to assemble.

I don't know if there's any rate limiting on that API.  If there isn't, then it wouldn't be super complicated to just make a series of requests and treat it like a geocoder.

### Forward Sortation Areas

The first half of a Canadian postcode is called the "Forward Sortation Area", and the boundaries of these are readily available public data.  The boundaries of electoral districts are also readily available, so they can be combined.  The limitation is that FSAs don't necessarily fit neatly inside a single electoral district, but many do and for those that don't we can at least get to a list of 2 or 3 possible districts.

#### Data sources

* FSAs and *federal* electoral districts are available from https://www12.statcan.gc.ca/census-recensement/2021/geo/sip-pis/boundary-limites/index2021-eng.cfm?year=21 .  A BC-only extract from this is stored here as `bc_fsas.gpkg`
* BC *provincial* electoral districts are available from https://elections.bc.ca/resources/maps/gis-spatial-data/ .  Note that this system doesn't allow for instant downloads; you have to make a request and it emails you later.  In my experience it doesn't take very long.  A copy of the 2024 electoral districts is stored here in the `bc_electoral_districts` folder with all the metadata it came packaged with.


