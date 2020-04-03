>![image](https://user-images.githubusercontent.com/3941856/78284119-c0d00080-74d3-11ea-88c6-23a22910be5d.png)

>My suggestions start between the end of "Mobile support" and the "How to contact us" section
_____

## REST API Quickstart

To get started with creating Daily.co API rooms, sign up for a free Daily.co account and head to the [Developers tab](https://dashboard.daily.co/?lstate=developers) in your dashboard. You will need your API key to create rooms at your Daily.co subdomain using the REST API.

![developer key page](https://p112.p2.n0.cdn.getcloudapp.com/items/GGuNGBOv/developer%20key.png?v=7696dfae57aac12918c32c7cd98429ef)

Once you have your key, you can make real API requests to create, update, list, and delete rooms and meeting tokens [right inside the documentation](https://docs.daily.co/reference#). 

![Daily.co try it](https://user-images.githubusercontent.com/3941856/78293746-f548ba00-74dd-11ea-9fd2-43145b9f7d72.png)

Click the Try It button and enter your API key to make requests and see the results in the right pane. Adjust the body parameters to your liking (or don't to create a room with the default settings). 

At the top of that right pane, you can see the code you can use to create the room with the settings you specified. 

![request using cURL ](https://user-images.githubusercontent.com/3941856/78391389-1f13e680-759b-11ea-8274-9b8684054a4f.png)

You can see examples in cURL, Node, Ruby, JavaScript, and Python.

This is an example of a request using cURL (with the API key omitted) to create a room with chat enabled

```bash
curl --request POST \
  --url https://api.daily.co/v1/rooms \
  --header 'authorization: Bearer *Replace with your API key*' \
  --header 'content-type: application/json' \
  --data '{"properties":{"enable_chat":true}}'
```

Check out our [blog post to get up and running in 5 minutes](https://www.daily.co/blog/embedded-video-calls-in-5-minutes-with-repl-it-and-daily-co) with publicly accessible embedded video calls.

