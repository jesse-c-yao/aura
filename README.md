# Before You Start

What matters most in modern AI-assisted development? What does the end goal look like?
I would say it comes down to two simple things: describe precisely what you want, and verify that the outcome matches what you want.

If we think a little further, those two things really come down to one: know what you want. Once you have described it precisely, you can leverage AI to put together all the test cases needed to verify that the outcome meets your expectations.

And that is exactly what this repository is for.

Before we dive into the details, note that the example here comes from mobile development, but the approach is intended to apply to all types of applications.

Let's get started. I hope you find it useful.


# Define Your App - App Document Model

The app document model is the single, validated data model for the app.
Visualize for Human is a read-only projection of that model.

See [App Document Model Design](app-docs/app-doc-design.md) for the proposed
structure, document responsibilities, reference rules, and validation strategy.


# Visualize for Human - App Aura

App documents need to be complete enough for AI to build an application, but
that alone is not enough. The harder question is: how can people be confident
that the documented product is the product they actually want?

App Aura answers that question by turning the canonical app document model into
an interactive preview that never modifies the source documents. Rather than
reading specifications in isolation, people can preview user journeys, inspect
screen details, and view the same application through perspectives designed for
different audiences.

See [App Aura Design](app-aura/app-aura-design.md) for the proposed system architecture,
runtime behavior, and functional scope.

A video demonstrate the experience with Aura.