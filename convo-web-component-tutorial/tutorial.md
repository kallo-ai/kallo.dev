# Kallo Convos Web Component Tutorial

<!-- TOC -->
* [Kallo Convos Web Component Tutorial](#kallo-convos-web-component-tutorial)
  * [Goals](#goals)
  * [Preparation](#preparation)
  * [Steps](#steps)
    * [Create a Topic](#create-a-topic)
    * [Configure the Topic](#configure-the-topic)
      * [Set the Model and Prompt](#set-the-model-and-prompt)
      * [Set the Sources](#set-the-sources)
    * [Obtain API Key](#obtain-api-key)
    * [Create Access Token](#create-access-token)
    * [Build an HTML Page](#build-an-html-page)
  * [Next Steps](#next-steps)
  * [FAQ](#faq)
<!-- TOC -->

## Goals

- Learn how to obtain an API key
- Learn how to curate the knowledge the AI will use in the convo using a topic
- Learn how to create access tokens
- Learn how to integrate the Kallo convo web component into a website

## Preparation

1. Have a Kallo account with a plan that includes API keys (either Advanced Team or an enterprise plan)
2. Download the files in the [sources](./sources) directory to your local computer
3. Get your favorite IDE or text editor ready (we're going to create an HTML file)

## Steps

As we go through the steps below, we will be gathering a few things along the way:

- [ ] Your user ID
- [ ] A new topic ID
- [ ] Your API key
- [ ] An access token (JWT format)

Keep these values handy as we locate/create them.

### Create a Topic

In the context of integrating Kallo into your own app, think of the topic as a single instance of an AI subject-matter expert. This single instance:

* Has its own set of configurable "knowledge"
* Can have its own AI system prompt to override default Kallo behavior (e.g. if you want to brand your own AI bot and give it guardrails)
* Is configured to use an LLM of your choice
* Is the single point of configuration for any number of convos your app instantiates in context of it

The convo web component will rely heavily on the topic you'll create in the Kallo app (this can also be done via the topics API). In fact, all the convos you instantiate with the web component will be topic-based convos. If you're not familiar with this concept, we'll touch on it below.

So let's create a topic:

1. Click "New Topic" in the [Topics](https://app.kallo.ai/topics) view: ![topics-view.png](./_images/topics-view.png)
2. Let's call this new topic **Future of AI**:![create-new-topic.png](./_images/create-new-topic.png)
3. Grab the topic ID from your browser's URL bar and keep it handy:![topic-id.png](./_images/topic-id.png)

### Configure the Topic

We now will change topic settings to ensure that convos served through your app behave in a certain way.

#### Set the Model and Prompt

1. Go to the topic *Settings* tab:![settings-tab.png](./_images/settings-tab.png)
2. Change the **Default AI Model** to *Claude 3 Haiku*
3. Add this message to the **Prompt** input: *You are a subject-matter expert on the future of AI. You are not allowed
   to discuss topics besides the future of AI. Only use the provided knowledge, or explain that you don't have the
   information.*

#### Set the Sources

1. Go to the topic *Sources* tab:![sources-tab.png](./_images/sources-tab.png)
2. Upload the PDF files contained in [sources](./sources)

These files will now become the basis of the AI assistant's knowledge.

### Obtain API Key

API keys are for the developer only and should never be compromised, shared, used in user-facing code, etc. API keys are discussed in detail [here](https://kallo.dev/docs/auth/api-keys). In this tutorial, we'll just provide the steps for generating them.

> ⚠️ Make sure your plan supports API keys. If it doesn't, you can upgrade to a plan that does in the [Account view](https://app.kallo.ai/account) or [contact us](https://kallo.ai/contact) to set up an enterprise plan without any limits (necessary for business deployments).

Steps:

1. In the Kallo app, navigate to the [Account view](https://app.kallo.ai/account) and find the API Keys section:
   ![img.png](_images/api-keys.png)
2. Click on "Create API Key" and then you will see the key displayed to you **one time only**:
   ![img.png](_images/api-key-created.png)

### Create Access Token

In production code for applications deployed at scale, your back end should generate access tokens for each user session.

> ⚠️ In this tutorial, we'll generate a long-term access token that can be reused, but it is **highly recommended** you generate a new access token every time you instantiate the convo web component instead.

Using the token from above, use Postman, Insomnia, cURL, or your other favorite method for making a REST call to Kallo's access tokens API.

We'll demonstrate with cURL below. Notice where we plugged in these values:

* API key: `h8rvjJE9sysTlSrZqb4I1qxpfKeZwHJ8GgQWlfgzJ9jaBoAVHfaOmJrcCXqLKqn0`
* User ID: `8d098657-3861-47a6-bdcb-28934d815d6a`
* Topic ID: `280ec90b-5984-45a7-a274-59303cc049c3`

Replace these values with the values you located or generated in the previous steps.

```shell
curl -X POST https://api.kallo.ai/access-tokens \
     -H "Authorization: Bearer api-key.h8rvjJE9sysTlSrZqb4I1qxpfKeZwHJ8GgQWlfgzJ9jaBoAVHfaOmJrcCXqLKqn0" \
     -H "Content-Type: application/json" \
     -d '
        {
          "userId": "8d098657-3861-47a6-bdcb-28934d815d6a",
          "scopeObjects": [
            {
              "id": "280ec90b-5984-45a7-a274-59303cc049c3",
              "type": "topic",
              "role": "participant"
            }
          ],
          "expiresAt": "2025-05-30T15:39:00Z"
        }
        '
```

Make sure you set `expiresAt` to be in the future.

And you should get a response that looks something like this:

```json
{
  "id": "70817a69-de67-47dc-ac0d-ba4edb09bcd8",
  "jwt": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9.eyJleHAiOjE3MTcwODM1NDAsImlhdCI6MTcxNzA4MzQ4NiwiaXNzIjoiYXBpLmthbGxvLmFpIiwianRpIjoiNzA4MTdhNjktZGU2Ny00N2RjLWFjMGQtYmE0ZWRiMDliY2Q4Iiwic2NvcGUiOiJ0b3BpYzoyODBlYzkwYi01OTg0LTQ1YTctYTI3NC01OTMwM2NjMDQ5YzMjcGFydGljaXBhbnQiLCJzdWIiOiI4ZDA5ODY1Ny0zODYxLTQ3YTYtYmRjYi0yODkzNGQ4MTVkNmEifQ.XYwmrRK8g4_hzqnJKpZzAfmurz9mjLvWVF6RT4UPhwbD65yiSK2ZkL-3aBbHg-Csrmbdfz9KZxdh7R7KtE7JtFYJLV4952RVW_KdIwbo0oJisMM7ypftH_K2QM_xCxhdkPc0VSrQI7K9x9PUromLncVmwk116_-8579WYNucoqctRs-WrqmLoaUIiTHoahJTYm8bUhnJRfPM9T56zvkuVn_oYolnS0P02Fj9ccNqA9tT1yUsZgmaZspL_h0QuCydCulkTU6FNusQTzYTDDdhZoUdwoJig2oougRtqZk2WLMsdjOJX-a8TIudhGJcjZX-0BNvFhuIsRxxBG9MrTlfkw"
}
```

The `jwt` value is the last value we needed on our checklist above.

### Build an HTML Page

Using [example.html](./example.html), look for these lines of code:

```javascript
element.systemConfig = {
  accessToken: "",
  userId: "",
  topicId: "",
};
```

1. Fill in `accessToken` with the `jwt` string we generated above
2. Fill in `userId` with your account's user ID
3. Fill in `topicId` with the ID of the topic we created earlier
4. Save the file
5. Open the file in a web browser

## Next Steps

This tutorial gets you started with convos in the simplest way possible. There are many possibilities for configuration of convos, including resuming past convos, convos that support user-specific context, and more.

Check out our evolving docs site, [kallo.dev](https://kallo.dev/). We also love talking directly to developers. For some personal 1-on-1 time with our engineers, reach out to us on our [contact form](https://kallo.ai/contact).

## FAQ

#### How do I configure styles in the component itself?

Check out our [configuration docs](https://kallo.dev/docs/web-components/configuration) for more info.

#### Can I see my visitors' convos?

Yes, they will appear in the Kallo app as convos within the topic.

#### In the convo, the user appears to be _me_. What the heck?

The main purpose of this tutorial is to show you how configuration of convos in your app works, generally. Therefore the user for which the JWT is generated above _is_ you. But, of course, you likely want to create a distinct user for your production use case. An enterprise plan is required to create dummy users or to create distinct users that are managed on Kallo's side. [Contact us](https://kallo.ai/contact) so we can get you set up with an enterprise plan that will provide you with this ability.

#### What are the usage limits?

If you are on a non-enterprise plan, usage is deducted from your plan; features that are "unlimited" when using the Kallo app may be limited when using API keys.

To avoid usage limitations and be allocated guaranteed capacity, an enterprise plan is required. Please [contact us](https://kallo.ai/contact) to set up an enterprise plan tailored for your use case.
