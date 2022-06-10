---
title: Platforms API Reference

toc_footers:
  - <a href='mailto:carter@checkinvestorstatus.com'>Request a Developer Key</a>
  - <a href='https://www.platformintegrationdemo.com/'>Live, Interactive Integration Demo</a>

search: true

code_clipboard: true

meta:
  - name: Platforms API Documentation
    content: Documentation for the Check Investor Status API
---

# Introduction

Check Investor Status provides our platform clients with an Accreditations API that allows them to access information about their investors' and sponsors' accreditation process.

There are 3 important parties:

- A **platform** is a website, company, or service that allows one or more sponsors to list investments to many investors. Common examples of platforms include 3rd-party administrators, crowdfunding companies, and investment listing services.
- A **sponsor** is a business that is fundraising from investors. This is interchangeable with the "issuer" of the security. Examples of this are real estate companies or startups in a fundraising round.
- An **investor** is a legal or natural person who is investing with a sponsor (often through a platform).

When a sponsor is raising funds under Regulation D Rule 506c, they are required to undergo "reasonable steps" to verify that each of their investors meet the legal definition of an accredited investor.

**Check Investor Status** verifies the accreditation status of investors. We help investors determine how they qualify and upload the proper documentation. We then have a licensed security attorney from our legal team review each investor's documentation, and issue a "Certificate of Accreditation" for that investor.
 
## Why use an integration?

An integrated platform experience allows you and your sponsors to automate, oversee, and manage your accreditation workflow.

**Integrating our accreditation product directly into your platform means:**

- Less friction for investors and less drop-off due to accreditation requirements.
- Less room for potentially-costly accreditation errors.
- Reduced cognitive overhead and manual work for your customer service, investor relations, and legal teams.
- Decreased spending on general or outside council.

**Highlights of the integration experience:**

- **Quick set up** – Set up is as simple as adding one API endpoint to your server and retrieving a URL for your investor. You control the UX of how the investor sees that button.
- **No Investor Registration** – You use our API to register your users as investors with Check Investor Status. This means your investors never have to create a Check Investor Status account.
- **No Investor Data Re-entry** – You can pre-fill investor information when you register (email address, type of legal entity, legal name) on your investors' behalf. They don't need to re-enter data you already have.
- **Automated Follow-up and Re-Rreview Processes** – Did an investor make a mistake in their documentation? Have they started but not finished their process? We'll followup with them and re-review when they fix their documentation.
- **Programmatic Access to Accreditation Info** – You can check the status of your investors as they proceed through the platform, or even download a PDF of the accreditation letter we issue for your accredited investors.
- **Ongoing Support and Development** – We're in active development of this product. Have a feature request? Not sure how something works? Just ask – if we haven't built it already, we can likely add support for it soon.

# Getting Started

Contact [carter@checkinvestorstatus.com](mailto:carter@checkinvestorstatus.com) to set up a platform integration account.

Once we've set up your platform account, you'll be given a **Platform API Secret Key** that you can use to access our platform API functions programmatically.

**Not sure how to proceed?**

If you have a use case and you're not sure if our API supports it, reach out to [Carter](mailto:carter@checkinvestorstatus.com). He can help advise your technical team on how to make the Check Investor Status platform integration work for you.

# Authentication

When you or your sponsors access accreditation data via the Accreditations API, you'll include your **Platform API Secret Key** as the request header `X-API-ACCESS` .

Your **Platform API Secret Key**  allows you to programmatically register investors, query investors' accreditation statuses, and more.

<aside class="notice">
You should treat your secret key as a master password. Anyone with it can access sensitive financial data about all of your investors.

You should never include your secret key in plaintext as code. For this reason, you should never hit this API from any user-facing application (EG a website or webapp). You should hit these endpoints from your own server, and should store your secret key as an environment variable in the server's runtime environment.

Check Investor Status is not liable for unauthorized access to the platform via your secret key.
</aside>

# The Investor Portal

<aside class="success">
Want to build your own accreditation portal? Starting in early Q3 of 2022, we'll be supporting a "Pure-API" solution that will let you handle the information intake without using our portal UX. Reach out to support with any questions or to be included in the beta for that workflow.
</aside>

We have a platform-specific User Experience built out that streamlines the process of investors uploading their documentation.

We dynamically generate secure links for investors to access their investor portals. These "portal links" allow you to provide your investors persistent access to their portal, without having them create accounts with Check Investor Status.

Note that, for security purposes, "portal Links" are expired and regenerated every time you look up a an investor with our API. 

It is not recommended to store a portal link in persistent storage (EG on your database). Instead, we highly recommend storing the investor's `investorId` on your database's representation of your user. With this, you can dynamically request a new portal link for the user when rendering the "Get Accredited" button.

## Return URLs

> Example adding returnUrl to portalLink

```jsx
// Assume we have portalLink available in scope already
// Encode the URL with JS builtin to make URL-safe
let urlSafeReturnURL = encodeURIComponent(
	'https://yourwebsite.com/some/investor/page?param=userCode'
);
// Updated link that will have users return to our site
let portalLinkWithReturn = `${portalLink}?returnUrl=${urlSafeReturnURL}`
```

Please note that you can attach a URL-encoded return url to any portal link. If you do this, then when the investor completes their process, they will be taken back to the return url you provide.

For example, let's say you fetch the portal link `https://checkinvestorstatus.com/platform/portal/42cce186ac4c6037a3a7815a26686ad3` from the API for your investor.

When your investor clicks that link, they'll be taken to their personalized investor portal. However, should they complete the accreditation process, you could have them redirect to your website (say `https://yourwebsite.com/some/investor/page?param=userCode`).

To do this, you'd URL-encode your return URL, and append it to the portal link as the URL search parameter `returnUrl`.

So your updated portal link would be `https://checkinvestorstatus.com/platform/portal/42cce186ac4c6037a3a7815a26686ad3?returnUrl=https%3A%2F%2Fyourwebsite.com%2Fsome%2Finvestor%2Fpage%3Fparam%3DuserCode`

If you do not pass a return URL, instead of being directed back to another place, the investor will be instructed that they can now safely exit the tab.

# Data Models 

## Investor Model

> Format of the user model

```json-doc
{
	// Investor Identifiers
	investorId: <a unique generated string identifying the investor>,
	creationDate: <Date the account was created>,
	
	// Investor Information
	email: <email address for the investor>,
	legalName: <legal name of the investor>,
	investorType: <entity type for the investor>,
	verificationMethod: <selected verification method (see below)>,
	metadataTag: <optional platform-supplied metadata for the investor>,

	// Status and Accreditation information
	status: <a status object for this user (see below)>,
	accreditations: [<An accreditation object for this user (see below)>],
}
```

An investor represents a single account in our API that can be an accredited investor. One user in your platform (eg. John Smith) may have multiple "investor accounts" at once – for example, if they are an individual investor, but also manage a company that invests.

An investor may be have multiple recorded accreditations. In fact, because accreditations expire, most repeat investors will have multiple accreditations associated with them.


### Platform-supplied fields (Pre-filled information)


Our API allows platforms to automatically pre-fill the `email`,  `legalName`, and `investorType` of their investors' accounts. However, the combination of those three fields must be unique – no two investor accounts can have the same combination of `(email, legalName, investorType)`.

### Investor Model Fields

An investor model has the following structure:


**`investorId`**

This is the unique identifier for an investor that you register with us.

We recommend that after you create an investor, you store their `investorId` with that account in your database. This can be especially helpful a single person ("Joe Smith") can have multiple investment accounts on your platform (EG "Joe Smith, the individual investor", as well as "Joe Smith, the signatory for Smith Family Offices").

**`dateCreated`**

Date when the investor's Check Investor Status account was registered.

**`email`**

The email address we will use to contact the user. They are automatically emailed when their accreditation status changes, or if we need updated / additional documentation.

**`legalName`**

Legal Names must correspond exactly with the legal name that appears on an investor's provided documentation. 

For example:

- An individual investor may have the legal name "Jane Doe"
- A married couple may have the legal name "Jane Doe & John Doe"
- An entity may have the legal name "Doe Family Investments, LLC"
- A trust may have the legal name "Doe Family Trust"

**`investorType`**

This represents the type of person or entity under which the investor is acting.

Investor type must be one of the following values:

Value | Description
--------- | -----------
`INDIVIDUAL` | a natural person investing as themself
`JOINT` | a married couple or spousal equivalent investing together
`EBP` | an employee benefit plan (retirement plan such as a 401k, IRA, etc.)
`ENTITY` | a business entity (EG an LLC) investing as a legal person
`TRUST` | a trust investing as a legal person

**`verificationMethod`**

Verification Method represents the qualification under which the investor qualifies as an "accredited investor." To learn more about the specifics of how this works, request a copy of our Regulatory and Compliance documentation.

The following are the possible values of `verificationMethod`:

Value | Description
--------- | -----------
`LETTER` | Individual qualified by letter from qualified professional
`INCOME` | Individual qualified by annual income
`FINANCE_PROFESSIONAL` | Individual qualified by professional financial qualifications
`DIRECTOR` | Individual qualified as a director at a sponsor
`EMPLOYEE` | Individual qualified as a knowledgeable employee for a deal
`NET_WORTH` | Individual qualified by their net worth
`NONE_OF_THE_ABOVE` | Individual qualified by extenuating circumstance
`JOINT_LETTER` | Spousal couple qualified by letter from qualified professional
`JOINT_INCOME` | Spousal couple qualified by annual income
`JOINT_NET_WORTH` | Spousal couple qualified by net worth
`JOINT_NONE_OF_THE_ABOVE` | Spousal couple qualified by extenuating circumstance
`EBP_NET_WORTH` | Retirement plan qualified by having assets in excess of $5,000,000 of value
`EBP_SELF_DIRECTED` | Retirement plan self-directed by one or more accredited individuals
`EBP_FIDUCIARY` | Retirement plan directed by a qualified plan fiduciary
`ENTITY_LETTER` | Entity qualified by letter from qualified professional
`ENTITY_NET_WORTH` | Entity qualified by assets under management
`ENTITY_STATUS` | Entity qualified by special regulatory status
`ENTITY_SPECIAL` | Entity qualified by combination of status and assets under management
`ENTITY_ACCREDITED_INDIVIDUALS` | Entity qualified as composed of solely accredited equity owners
`ENTITY_NONE_OF_THE_ABOVE` | Entity qualified by extenuating circumstances
`TRUST_LETTER` | Trust qualified by letter from qualified professional
`TRUST_NET_WORTH` | Trust qualified by assets under management
`TRUST_ACCREDITED_INDIVIDUALS` | Trust qualified as being composed solely of accredited "Equity Owners" (SEC's definition here is wonky, see Regulatory Compliance documentation for more details)
`TRUST_NONE_OF_THE_ABOVE` | Trust qualified by extenuating circumstances

**`metadataTag`**

This is a metadata string you can store on the investor that will be returned with the investor, as well as with billing events associated with them.

The most common use case is storing a sponsor identifier, so that you can track which sponsors to bill for which events. For example, if an investor is clicks "Get Accredited" on the invest page for a particular investment, you may add an identifier for that investment or sponsor as the `metadataTag` for that user.

This field is managed entirely by the platform – we provide you with the ability to set and update this field, but its use and management is up to you.


## Status Model

> Format of the Status Model

```json-doc
{
	status: <A Status Value (see below),
	expiration: <Date when the status expires, or null>
}
```

> Status Model Examples:

```json-doc
// An in-progress investor's status
{
	status: 'ACTION_NEEDED',
	expiration: '2022-03-20T07:00:00.000Z',
}

// A currently-accredited investor's status
{
	status: 'VERIFIED',
	expiration: '2022-05-20T07:00:00.000Z',
}

// An investor's status whose application is in review
{
	status: 'IN_REVIEW',
	expiration: null,  // Legal review doesn't expire
}
```

The status model represents where the investor currently is in their accreditation process.


**`status`** 

Where the investor is in the accreditation pipeline. Possible values are:

Value | Description
--------- | -----------
`NOT_FOUND` | We did not find an investor associated with that email in our records.
`ACTION_NEEDED` | The investor has started the accreditation process but has not completed it.
`IN_REVIEW` | The investor has uploaded documentation that is under legal review by our team.
`MORE_INFO` | We've reached out to the investor for more information or additional documentation.
`WAITING` | The investor accreditation is currently blocked on other accreditations (EG an entity waiting on the accreditations of its equity owners)
`ACCREDITED` | The investor has a current, valid accreditation. 
`EXPIRED` | The investor has a current, valid accreditation. 
`FAILED` | We have determined that we are unable to verify this investor's accreditation

**`expiration`** 

If the investor is currently accredited, this field will contain the date that the current accreditation expires. 

If the investor is not accredited, this field will contain the date their current status expires and their investor account is archived. Some statuses do not expire, but we can only hold information on an investor for 90 days.




## Accreditation Model

> Format of the Accreditation Model

```json-doc
{
	accreditationId: <Internal identifier of the accreditation>,
	verificationMethod: <Selected verification method>,
	dateCreated: <Date of creation of the accreditation>,
	expirationDate: <Date the letter expires>,
}
```

> Accreditation Model Examples:

```json-doc
// A current Accreditation (until 6/14/2023)
// Note that a real accreditation wouldn't expire
// so far out from when it was issued.
{
	accreditationId: "623139b30bca64b9298bc746",
	verificationMethod: "ENTITY_NET_WORTH",
	dateCreated: "2022-03-16T01:13:23.040Z",
	expirationDate: "2023-06-14T01:13:23.040Z",
}

// An expired Accreditation (as of 3/4/2022)
{
	accreditationId: "61e7b43c10a0f6e3830d0512",
	verificationMethod: "FINANCIAL_PROFESSIONAL",
	dateCreated: "2022-01-19T06:48:28.761+00:00",
	expirationDate: "2022-03-04T06:13:38.761+00:00",
}
```

The accreditation model represents an accreditation successfully completed through our platform.

**`accreditationId`**

The unique identifier for a particular accreditation. You will need this to download the PDF of the relevant accreditation letter.

**`verificationMethod`**

The method of verification used for this particular accreditation. The values are the same as the corresponding field on the investor model.

**`dateCreated`**

The date the accreditation was issued or created

**`expirationDate`**

The date this accreditation expires, and can no longer be used to verify the accredited status of the investor.




## Event Model

> Format of the Event Model

```json-doc
{
	date: <Date of creation of the event>,
	type: <String with the type of event has occurred>,
	investorId: <ID of the investor who the event pertains to>,
	metadataTag: <Any metadata stored on the investor>,
}
```

This represents an accreditation event that has occurred. We log all events, and deliver them to you via a webhook that you can configure. Generally, we send events when either:

1.  An investor's status changes,
2.  A billable event occurs, or 
3.  An accreditation process is completed (successfully or otherwise)


The types of events you may receive are:

Type | Description
--------- | -----------
`INVESTOR_REGISTERED` | an investor account has been registered with our platform
`INVESTOR_STARTED` | an investor has begun their accreditation process (fires when the investor selects their accreditation method)
`SENT_TO_REVIEW` | an investor accreditation application has been sent to legal review (either initially or after having more information requested)
`MORE_INFO_REQUESTED` | our legal team has requested followup info from an investor who submitted their application
`BILLABLE_EVENT_FULL_REVIEW` | an investor's accreditation application has been completed and sent to legal review
`BILLABLE_EVENT_LETTER_VERIFICATION` | an investor's existing accreditation letter has been sent to our team to be processed and verified
`ACCREDITATION_SUCCEEDED` | an investor's accreditation application was approved, and an accreditation letter was issued
`ACCREDITATION_FAILED` | an investor's accreditation application was denied or expired due to inactivity

To set up your webhook receiver, use the `platform/admin/update-webhook-url` endpoint (documented below).

We currently do not retry delivery on webhook events that are not delivered successfully, but you can retroactively pull a list of your events from the last month with the `platform/events/query` endpoint (documented below).

Note that we use the `BILLABLE_EVENT_*` events to generate your monthly invoice. You can use these events to track or compute what your bill will be.

# API Endpoints

We provide API access to read and modify your investors' accreditation data programmatically.

Some common use cases for the Accreditation API include:

* Registering your users as Investors in our platform.
* Fetching a portal link for your user to click.
* Querying all of your investors, or events that have occurred.
* Getting the accreditation or status associated with a user in your platform.
* Creating custom dashboards for your sponsors or internal teams.
* Updating your webhooks or querying events that were delivered to your webhooks in the past.


## Register an Investor

To create new investors on our platform, you will use the `register` endpoint.

This endpoint allows you to register an investor for your platform. If an investor matching the criteria you mentioned exists already, that investor will be returned.

This endpoint also allows you to pre-fill information about the investor (so they will not be asked to re-enter that information).

### HTTP Request

`POST /platform/investors/register`

### Request Body Parameters

Parameter | Required | Description
--------- | ------- | -----------
email | true | a valid email address of an investor in your platform
legalName | false | a string containing the exact legal name of the investor in your platform
investorType | false | an `investorType` value representing the type of entity the investor will be investing as. Valid values are `INDIVIDUAL`, `JOINT`, `ENTITY`, and `TRUST`
metadataTag | false | any string value you want to store as metadata on the user (EG – the id of their sponsor). This value is always returned with the user, and is passed along with billing events
equityOwners | false | an array of objects representing the equity owners for this entity, trust, or EBP. If you know this equity-owner information up front, you can use this field so that your investor doesn't have to re-enter the information themselves. The format of an equityOwner is `{ email, legalName, investorType, isSigningInvestor }` – where the first three fields are the same as described above, and the `isSigningInvestor` field is a boolean set to `true` if that equity owner represents the same person who is filling out the accreditation on behalf of the entity.

### Returns

> Return format

```json-doc
{
	new: <true if new investor created, false otherwise>,
	investor: <Investor model that was found or created>,
	portalLink: <An updated portalLink for the investor found>,
}
```

If there is already a user matching the parameters provided, this endpoint will return that user instead of creating a new one. If there are multiple users matching the parameters provided, the endpoint will return a 400 error with the message "ambiguous registration – more specificity required".

### Notes

> POST body for Joe's Entity account

```json
{ 
	 "email": joesmith@gmail.com",
	 "investorType": "ENTITY",
	 "legalName": "Smith Family Investment, LLC" 
}
```

> POST body for Joe's second entity account

```json
{
	"email": joesmith@gmail.com",
	"investorType": "ENTITY",
	"legalName": "New Investment Company, Inc." 
}
```

> POST body for an entity with two equity owners. Note that this is registering for an entity – Jackson Investments. In this case, you know that the entity has two accredited owners – Jackson Smith and Jessica Ramirez. You can pre-fill the entity owner information so that the signer – Jackson in this case – doesn't have to reenter it as evidence during the entity's accreditation process.

```json
{
	"email": jackson.investments@gmail.com",
	"investorType": "ENTITY",
	"legalName": "Jackson Investments, Inc.", 
	"equityOwners": [
		{
			"email": "jackson.investments@gmail.com",
			"investorType": "INDIVIDUAL",
			"legalName": "Jackson Smith",
			"isSigningInvestor" true
		},
		{
			"email": "jessicaramirez@gmail.com",
			"investorType": "INDIVIDUAL",
			"legalName": "Jessica Ramirez"
		}
	]
}
```

Although we only require an email-address for the user, we recommend that you make your query as specific as possible.

For example, let's say Joe Smith has a single account on your platform associated with `joesmith@gmail.com` . With that account, he can invest either as himself, or as his family office "Smith Family Investment, LLC".

If you were to hit the registration endpoint with: `{ "email": "joesmith@gmail.com" }` as your post body, that registration attempt already matches two investor accounts: 
1. Their personal investor account, and 
2. Their family office investor account. 

In this case, the endpoint would send an error so as to avoid sending back the wrong account. 

To fix this problem, you could use the POST body in the example on the right to resolve any ambiguity.

If Joe wanted to start another entity account to represent his managing a different investment company ("New Investment Company, Inc."), you could register that account as shown in the example's second post body.


Note that metadata is not included in the "specificity" parameters. If you try to register an investor that already exists, but with a different metadata tag, the existing investor will be returned with the old metadata tag. To update a user's metadata, you will want to use the `/platform/admin/update-metadata/:investorId` endpoint.


## Query your Investors

> Request Body Format

```json-doc
{
	"pageSize": <integer number of results to return, default 50>,
	"pageNumber": <integer number of the page to return, default 0>,
	"investorIds": [<investorIds to include>],
	"emails": [<emails to include>],
	"filterToStatus": <status string to include>,
}
```

> Return Format

```json-doc
{
	"count": <number of results returned>,
	"investors": [<Investor Objects>],
}
```

> Examples of `/query` request bodies 

```json-doc
// Look up your 25 most recently created investors
{
	"pageSize": 25,
	"pageNumber": 0
}

// Look up your 10 most recently created investors who are accredited
{
	"pageSize": 10,
	"pageNumber": 0,
	"filterToStatus": "VERIFIED"
}

// Look up a list of specific users
{
	"investorIds": [
		"sandbox-platform-investor-1aa64ab0e13cc6c919cf40effddb585813705765",
		"sandbox-platform-investor-c2bc40325bd1283f21da4852f00c6d77e073cbdc"
	]
}

// Get all currently-approved investors on your platform
{
    "filterToStatus": "VERIFIED"
}

// Get any investors in a specific list with an "ACTION_NEEDED" status
{
    "investorIds": [
        "sandbox-platform-investor-1aa64ab0e13cc6c919cf40effddb585813705765",
        "sandbox-platform-investor-c2bc40325bd1283f21da4852f00c6d77e073cbdc"
    ],
    "filterToStatus": "ACTION_NEEDED"
}

// Get all investor accounts associated with a single email address
{
    "emails": ["sample-investor@example.com"],
}
```


This endpoint lets you run queries against all of your platform's investors.

This optionally allows you to request the investors associated with a particular set of email addresses or investor ids. It also allows you to filter the results to those with a particular status.

Note that this approach does not create (or return) new portal links for the returned investors.

Investors are returned in order of creation date (newest first).

This is especially useful for building out internal and sponsor-facing dashboards on your platform.

### HTTP Request

`POST /platform/investors/query`


### Request Body Parameters

**Pagination**

`pageSize` and `pageNumber` allow you to paginate your search results. Note that your pages are 0-indexed – your 50 most recent results are on page 0.

By default, we will return the first page, using a default value of 50 results per page.

### Returns

The query endpoint returns the number of results returned, as well as a list of the investors found by the query.


### Examples

See the code tab to the right for examples of Query post request bodies, and descriptions of what each would return.


## Convenience Endpoints

With just `register` and `query`, the platform API is functionally complete. However, there are a few convenience endpoints for fetching information about your investors by `investorId`. 

Use of these endpoints is a one reason to store your platform users' `investorId` in your database alongside their other account information.

---

### `GET /platform/investors/lookup/:investorId`

> Return format of `/platform/investor/lookup/:investorId`

```json-doc
{
	investor: <Investor Object>,
	portalLink: <updated portal link for your investor>
}
```

Fetches all of an investor's accreditation information by their `investorId`.

Note that this returns the investor object, as well as an updated portal link.

**Request Params**

Parameter | Description
--------- | -----------
investorId | The investor Id of an investor from your platform

**Returns**

A json object with the investor object and an updated `portalLink` for that user.

---

### `GET /platform/status/lookup/:investorId`

> Return format of `/platform/status/lookup/:investorId`

```json-doc
{
	status: <Status Object>,
}
```

Fetches the status of an investor by their `investorId`.

**Request Params**

Parameter | Description
--------- | -----------
investorId | The investor Id of an investor from your platform

**Returns**

A json object with the current status of this investor.

---

### `GET /platform/accreditations/lookup-current/:investorId`

> Return format of `/platform/accreditations/lookup-current/:investorId`

```json-doc
{
	currentAccreditation: <Accreditation Object, null if none current>,
}
```

**Request Params**

Parameter | Description
--------- | -----------
investorId | The investor Id of an investor from your platform

**Returns**

A json object with the current accreditation for an investor, if there is one. Otherwise, `null`.

---

### `GET /platform/accreditations/lookup-all/:investorId`

> Return format of `/platform/accreditations/lookup-all/:investorId`

```json-doc
{ 
	accreditations: [<Accreditation Object>],
}
```

**Request Params**

Parameter | Description
--------- | -----------
investorId | The investor Id of an investor from your platform

**Returns**

A json object with an array of all accreditations (past and present) for this particular investor. No order is guaranteed.


## Download an accreditation letter

Download an accreditation letter as a PDF.

### HTTP Request

`GET /platform/accreditations/download/:accreditationId`

### Request Params

Parameter | Description
--------- | -----------
accreditationId | The accreditation ID for an accreditation

### Returns

Downloads a PDF of the associated accreditation letter.



## Admin and Event Endpoints

There are a few simple endpoints for administrative tasks – testing your connection, setting metadata tags on investors, and working with our webhooks.

---

###  `GET /platform/admin/ping`

Tests your connection with our API, returning `{ success: true }` if your API secret key has successfully connected you to the Accreditations API.

This is ideal for testing out your authentication.

---

### `POST /platform/admin/update-metadata-tag/:investorId`

> Format of the post body

```json-doc
{
	"metadataTag": <new metadata tag value as a string>
}
```

> Return format for `/platform/admin/update-metadata-tag/:investorId`

```json-doc
{
	investor: <Investor Object>
}
```

Update an investor's `metadataTag` property.

You can remove an investor's current metadata tag by passing an empty string or null value as the `metadataTag` field's value in the post body.

**Request Params**

Parameter | Description
--------- | -----------
investorId | The investor Id of an investor from your platform


**Returns**

A json object with the updated investor object.

---

### `POST /platform/admin/set-webhook-url`

> Format of the Post request body delievered to your `webhookUrl`

```json-doc
{
	event: <Event object>
}
```

> Format of the post body

```json-doc
{
	"webhookUrl": <new url to which we post webhook events>
}
```

Updates your platform's webhook URL for recieving events.

If there is no webhook url set, we will not deliver events. We will still keep record of them on our servers.

The url you set up on your API should be able to accept a POST request that accepts a JSON object with an event object formatted as such

---

###  `POST /platform/events/query`

> Format of the post body

```json-doc
{
	"pageSize": <integer number of results to return, default 50>,
	"pageNumber": <integer number of the page to return, default 0>,
	"filterToType": <Event Type to include>,
}
```

> Return format of `/platform/events/query`

```json-doc
{
	"count": <number of results returned>,
	"events": [<Event Objects>],
}
```

Pulls a backlog of your accreditation events.

You can optionally specify the type of event that you're looking for. This will return events whether or not they have been successfully delivered to your webhook receiver.

Events are returned in order of creation date (newest first).


**Request Body Parameters**

`pageSize` and `pageNumber` allow you to paginate your search results. Note that your pages are 0-indexed – your 50 most recent results are on page 0.

By default, we will return the first page, using a default value of 50 results per page.

> Some sample queries. These JSON objects represent the body of the POST request.

```json-doc
// Look up your 25 most recently created events
{
	"pageSize": 25,
	"pageNumber": 0
}

// Look up the events for your last 10 failed accreditations
{
	"pageSize": 10,
	"pageNumber": 0,
	"filterToType": "ACCREDITATION_FAILED"
}
```


# Integration Example

## Providing your users with a "Accreditation Portal" Button

> Your user or account model (in your database)

```json-doc
{
	"_id": <internal database id for user>,
	"email": "johndoe@example.com",
	"name": "John Doe",
	"businessType": "INDIVIDUAL",
	"sponsor": "ABC Investments",
	"checkInvestorStatusId": <userId from our platform, maybe null>,
}
```

We can put this all together in a comprehensive example by demonstrating how we could provide a user on your platform with a button that links them to their user portal.

**You can see this code in action:**

- This example lives in a live, interactive demo at [PlatformIntegrationDemo.com](https://www.platformintegrationdemo.com/). 
- The front-end code we write in this example lives at [this public repo](https://github.com/carterjbastian/my-investment-platform)
- The back-end code we write in this example lives at [this public repo](https://github.com/carterjbastian/my-investment-platform-api)

For this example, let's assume that your platform is named **MyInvestmentPlatform**. In your database, you have an account model that looks as formatted in the snippet on the right.

We'll be implementing your user-facing front-end in as a react-based web application, and we'll be implementing your web server as endpoints in a node/express API. However, this general approach and structure can be taken regardless of platform, language, or infrastructure.

### The overall information flow is as follows:

1. Your user opens a page on your front end webapp 
2. Your webapp requests a `portalLink` from your web server.
3. Your web server hits the `/platform/investors/register` of the Check Investor Status API and returns the `portalLink` for the new investor.
4. Your webapp uses the portalLink as the `href` in the rendered "Verify Accreditation" button.

This may seem like a lot of steps, but it's necessary to have your server act as an intermediary so that your users don't need to create accounts on our platform.

Let's break down each step and provide code snippets.

## 1. Your user opens a page on your front-end webapp

This one is somewhat self-explanatory. This could be any page – a "My Account" page, an investment portal, etc. 

You can see an example of such a page in step two of the [live demo](https://platformintegrationdemo.com/).

## 2. Your webapp requests a portal-link from your web server

> An example component requesting a portal-link to create a button

```jsx
/* Inside UserFacingComponent.jsx */
// Save the portalLink from Check Investor Status as component state
const [portalLink, setPortalLink] = useState('');

// Get the user's ID from app context
const [{ userId }] = useContext(AppContext);

// Load in the user from the API
useEffect(async () => {
  const { 
	error, 
	portalLink: newPortalLink, 
  } = await getPortalLinkFromAPI(userId);

  // Save the token in localstorage and in the app state
  if (error) {
    // Handle Error 
  } else {
    // Set the portalLink state in the component
    // With the response from the api
    setPortalLink(newPortalLink);
  }
}, []);
```

> The `getPortalLinkFromAPI` function may look something like this:

```jsx
/* getPortalLinkFromAPI function */
async function getPortalLinkFromAPI(userId) {
  return fetch(
	`https://yourservice.com/api/portal-link/${userId}`,
	{
      method: 'Get',
      headers: {
        'Content-Type': 'application/json',
      },
    })
    .then((data) => data.json())
    .catch((err) => ({ error: err }));
}
```

You can do this however you generally have your front end and back end interact. In the example on the right, we use React Webhooks.

In this example, your backend web server is located at [https://yourservice.com]().

Note also that your API is likely protected by some sort of authentication, but this is left out of the example for simplicity.


## 3. Your web server hits the `/platform/investors/register` endpoint of the Check Investor Status API

```javascript
/* api/portal-link.js (in webserver living at yourservice.com) */

const axios = require('axios').default
const express = require('express')
const app = (module.exports = express())

const { getUserFromDatabase } = require(/*...*/)

app.get('/api/portal-link/:userId', async (req, res) => {
  const { userId } = req.params
  // Get your user from your database
  const {
	name,
	email,
	businessType,
	sponsor,
  } = await getUserFromDatabase(userId)
  
  try {
	let response = await axios.post(
	  `https://check-investor-status.herokuapp.com/platform/investors/register`,
	   // Post Body
      {
        // Map your internal user data to CI
        // parameters in the request body.
        email,
        legalName: name,
        investorType: businessType,
        // Optionally store the sponsor as metadata
        metadataTag: sponsor,
      },
      // Headers including authorization
      {
        headers: {
          'Content-Type': 'application/json',
          // Your secret key lives in your environment variables
          'X-API-ACCESS': process.env.CI_API_SECRET_KEY,
        },
      },
    )
    // Deconstruct the response
    let {
      new,  // Whether investor was created (vs. found)
      investor,  // Investor model
      portalLink,  // What we care about
    } = response.data

    // Return portalLink the front end
    return res.send({ portalLink })
  } catch (err) {
    return res.status(500).send(err)
  }
})
```

So now we just have to mock up the endpoint on yourservice.com (your back end). Let's assume that you have your API secret key stored as the environment variable `CI_API_SECRET_KEY`.

Your endpoint may look something like the express endpoint on the right.


## 4. Your webapp uses the portal-link as the `href` in the rendered button.

```jsx
/* Inside UserFacingComponent.jsx */
const UserFacingComponent = function () {
  const [portalLink, setPortalLink] = useState('');

  // Get the user's ID from app context
  const [{ userId }] = useContext(AppContext);

  // Load in the user from the API
  useEffect(async () => {
	  // Code from above (Step #2)
	  // ...
  })

  // Render a link for the user
  return (
    <a
      href={portalLink}
      target="_blank"
      className='link'
    >
      Accreditation Portal
    </a>
  )
}
```

Finally, now that we've used an API call to yourservice.com, which in turn hit the Check Investor Status API, we should have access to it in the component we started above.

Specifically, the `useEffect` hook in step 2. will store the portalLink specific for your user to the `portalLink` state.

Now we can render that however we please – a link, and . If the user visits the `portalLink`, they will automatically be taken to Check Investor Status' platform experience. They'll be logged in (no account creation) and their account information will be pre-filled (no information re-entry).


### That's it!

That alone is sufficient to provide your users with access to the entire Check Investor Status accreditation process.

Other endpoints can be hit with a similar data flow. Using the other documented endpoints, you can do everything from checking the verification status for a user to building dashboards for sponsors.


