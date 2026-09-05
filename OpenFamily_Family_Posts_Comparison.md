# OpenFamily Community Enhancement — Family Posts

## Package 4 — New Feature Addition

This package adds a new **Family Posts** feature to OpenFamily.

This is a new capability rather than a correction to existing functionality.

## Purpose

Family Posts gives household members a private shared area for information that does not naturally belong in Calendar Events, Tasks, Shopping, Planning, Recipes, or Family Notes.

Family members can use it for personal updates, photos, useful links, or anything else they want to share with the household.

## Added functionality

### Family Posts

A new **Posts** section is added to OpenFamily.

Family members can:

- create personal text posts
- share photos
- share web links
- edit their own posts
- delete their own posts
- see when posts were created or edited

### Seen acknowledgements

Posts include a Seen system.

Family members can mark a post as Seen, and the post shows which family members have viewed it.

This provides a simple way to know that a shared family update has been noticed without requiring a separate reply.

### Today integration

The most recent Family Post is displayed directly in the **Today** section.

This allows current family updates to remain visible without requiring users to open the full Posts page.

Unseen posts receive additional visual emphasis.

### Real-time updates

Family Posts uses OpenFamily's existing WebSocket update system.

Creating, editing, deleting, or acknowledging a post can therefore update other connected family clients without requiring a manual page reload.

### Mobile and desktop

The Posts interface is designed for both desktop and mobile layouts.

A **New Post** option is also added to the existing mobile quick-action menu without altering the five-item mobile bottom navigation.

## Data model

The feature adds:

- `family_posts`
- `family_post_seen`

Posts retain both the shared family scope and the individual signed-in account that created the post.

This allows posts to remain shared with the household while restricting editing and deletion to the original author.

## Images

Photos are resized in the browser before being submitted and stored with the Family Post.

The implementation does not require a separate public upload directory.

## Security and privacy

The public enhancement package excludes:

- personal posts
- personal photos
- credentials or API keys
- runtime family data
- deployment configuration
- backup files
- server-specific information

Only the functional source changes required for Family Posts are included.

## Package files

- `openfamily-family-posts.patch`
- `openfamily-family-posts-files.txt`

## Compatibility

This package was built and tested against the customized OpenFamily installation represented by the preceding community enhancement packages.

It is provided as a tested reference implementation and should not be assumed to apply automatically to a newer OpenFamily `main` branch without review.
