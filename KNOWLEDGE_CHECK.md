# Lab 09: Knowledge check

**Answer each question below it, in your own words, in one to three sentences.** Replace the `...` below **Your answer:** with your text.

## 1. Credentials in the image?

Assume you want to share your image with others and make it publicly available on `ghcr.io`, so that anyone can pull it and run the analysis. A teammate suggests adding `COPY .aws /home/app/.aws` to the `Dockerfile`, so that nobody has to type `-v ~/.aws:/home/app/.aws:ro` any more. (This is a thought experiment: keep your own package private.)

**Your answer:** Give two reasons why this is a bad idea. Think of who could then read your credentials, and of what happens when your Learner Lab session ends.

The first reason why this is a bad idea is because the .aws folder has my specific aws credentials (it has the files credentials and config); so, if my aws credentials are included in the image and then the image is made publicly available on 'ghcr.io', anyone can pull the image and read my credentials. With the keys listed in my credentials, they can act as me in my aws account and tamper with my data (and all following charges will be billed to me).

Another reason why this is a bad idea is because the credentials (given by the learner lab) are only temporary. After a few hours, the credentials will expire, but the image will not update the credentials because they are only made when the image is built.

These two reasons are the main reasons why .aws/ was included in the .dockerignore file in lab 9.

## 2. Where did the Atlas password come from?

In Step 4, you ran the analysis once with `-v ~/.aws:/home/app/.aws:ro` only, without the three `-e` options. The container still wrote to your Atlas cluster.

*Hint:* Read `get_atlas_settings()` in `src/book_analysis/analyze.py`.

**Your answer:** Explain, step by step, where the container found the Atlas settings in that run.

1)  The container starts after the `-v ~/.aws:/home/app.aws:ro` call.

2)  Docker runs the `analyze.py` file because of this line in the Dockerfile: `ENTRYPOINT ["python", "-m", "book_analysis.analyze"]`

3)  The `main()` function in the `analyze.py` file calls the `connect()` function which calls `get_atlas_settings()`.

4)  Following the first `if` block in `get_atlas_settings()`, the function tests "MONGODB_ATLAS_URL" (it doesn't get to the other two because it fails to find the first of the three, which is "MONGODB_ATLAS_URL") in os.environ and doesn't see them because the initial command omitted the three `-e`.

5)  Still inside `get_atlas_settings()`, it loads credentials from the SSM parameter set in aws (this was set in an earlier class activity/lab). It can only access these because of the credentials set in the `credentials` file in `.aws`.

6)  AWS authenticates the request and returns the parameter (SSM returns it as a JSON string).

7)  (Still inside `get_atlas_settings()`) `json.loads(resp["Parameter"]["Value"])` turns that string into a dictionary with the url, user, and password, in the same shape the environment-variable branch would have returned.

## 3. Same tag, different image

Run `docker images book-analysis`, and note the image ID of `book-analysis:0.1.0`. Then change the `description` in the `LABEL` of your `Dockerfile`, build again with the same command, `docker build -t book-analysis:0.1.0 .`, and run `docker images book-analysis` again.

**Your answer:** Paste the image ID from before and after the rebuild. Why is it a problem for someone who uses your image that `0.1.0` now names a different image, and what should you have done instead?

Image ID before the rebuild: bfb2f485a278

Image ID after the rebuuild: 7d3ff90668dc

It's a problem that the ID changed after the rebuild because someone who pulled the image before the rebuild got a different image than someone who pulled it after the rebuild, but there is no indication to either of them that there was an update of some kind. The 0.1.0 tag was overwritten, and you can't get the "old 0.1.0" back.

Instead, you're supposed to have treated a published version tag as immutable, meaning that if you need to make an edit after you've built an image, you should use a different tag, such as `docker build -t book-analysis:0.1.1`.
