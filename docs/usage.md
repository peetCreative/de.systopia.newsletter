# Usage

Using this extension is dependent on how you implement the forms in your
external system. This guide is therefore somewhat generic in explaining the
general workflow of subscriptions.

In order to allow users to manage their preferences instead of simply
unsubscribing, you should not use CiviCRM default tokens when sending a mailing
but the custom token provided by the extension that links users to their
personal preferences page. The token for the default profile is
`{newsletter.preferences_url_default}` but you will be able to choose the token
for the profile you would like to link to from CiviCRM's token menu.

![Token Menu](img/token_menu.png?raw=true "Token Menu")


## Workflow

This extension assumes a form-based flow of users subscribing to mailing lists,
confirming their subscription and possibly changing them at a later stage.

- The user first subscribes to one or more mailing lists. They will be matched
  with existing contacts and receive a confirmation/double opt-in email with a
  link to their confirmation/preferences page.
- After following the link and confirming and/or changing their preferences,
  they will be added to/removed from groups according to their choices. An
  information email will be sent to them automatically.
- In order to change their preferences, the user needs to use a personalized
  link. That link can be included in CiviCRM mailings via the tokens provided by
  the extension or by your external system. The link needs to contain the
  contact's checksum and the identifier for the profile being used.

### Subscription stage

The user is presented a form where they can subscribe to mailing lists by
selecting them from a given set of options, and provide personal data for them
to be identified by CiviCRM.

Which mailing lists are available as options and which personal data fields are
available and/or required, is dependent on the profile.

The form input is sent in to the `NewsletterSubscription.Submit` API action that
this extension provides. The API action will validate the input (a valid
profile, required contact data), identify or create the contact, add group
memberships as *Pending* for newly subscribed mailing lists (the contact may be
subscribed to them already), and generate and send a confirmation e-mail to the
contact, containing a link to a personalised preferences form, which is used for
confirming and changing subscriptions.

### Confirmation/Preferences stage

The user follows their personalised preferences link they have received in the
subscription stage and is presented a form where they can confirm and change
their subscriptions by (de-)selecting mailing lists, similar to the previous
stage.

When generating this form, the system issues a request to the API action
`NewsletterSubscription.Get` that this extension provides. Taking the profile
name and the contact checksum, this action retrieves the subscription status for
all mailing lists and the contact data for all contact fields defined within the
given profile, and sends it to the system which, in turn, creates the
preferences form based on that information.

The form input is sent in to the `NewletterSubscription.Confirm` API action that
this extension provides. The API action will validate the input and update all
group memberships for the configured mailing lists, setting them to either
*Added*, when the user selected it, or *Removed*, when the user de-selected it.
The form will then generate and send an information e-mail to the contact, again
containing a link to their personal preferences form for further reference.

Also, the form might issue a request to the API action
`NewsletterSubscription.Confirm` with the `autoconfirm` parameter set, prior to
presenting the form to the user, to opt-in the user without them actively
submitting the form. This might be useful to avoid *Pending* subscriptions due
to users not submitting the confirmation form.

This stage acts as a double opt-in functionality, which is legally required in
some circumstances and replaces CiviCRM's double opt-in e-mails.

Also, the pereferences form may be used multiple times by the contact to change
their newsletter subscriptions without having to enter their contact data again.

### Request stage

Since the link to the user's preferences page is only being distributed via the
e-mails that this extension sends when subscribing or confirming subscriptions,
the user might want to re-claim that link in case they deleted the e-mail(s).

The external system may provide a form for requesting the information e-mail by
any contact.

When generating this form, the system issues a request to the API action
`NewsletterProfile.Get` and provides the user with all contact fields defined
within the profile, but no mailing list options. Instead of presenting a form
for the user to input their contact data again, a link may be provided to
request the API to re-send the confirmation e-mail (containing a link to the
preferences page) by providing the contact checksum and the contact ID (which is
to be retrieved using the contact checksum).

The form input is sent in to the `NewsletterSubscription.Request` API action,
which will generate and send the information e-mail to the contact as in the
Preferences stage.


## Specific implementations

The general workflow of this extension needs to be implemented by the external
system (e.g. a website). Listed below are known implementations that are ready
to use on the particular system types. Let us know if you have developed another
implementation and want it to appear in this section.

### Drupal module

There is a module for the [Drupal](https://drupal.org) Content Management System
available, named
[Advanced Newsletter Management](https://github.com/systopia/civicrm_newsletter/).

It provides all three forms (subscription, preferences, request link) and was
developed in tandem with this extension.

It is dependent on the [CiviMRF](https://drupal.org/project/cmrf_core) library
module for making standardised requests to any CiviCRM REST API over the web or
locally (when running CiviCRM and the Drupal Website in the same installation).
