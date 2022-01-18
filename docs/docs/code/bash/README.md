# Writing Bash in Code Steps

Prefer to write quick scripts in bash? We've got you covered.

You can run any bash in a Pipedream step within your workflows.

## Adding a Bash code step

1. Click the + icon to add a new step
2. Click "Custom Code"
3. In the new step, select the bash runtime in language dropdown

## Logging & debugging

When it comes to debugging bash scripts, `echo` is your friend.

Your `echo` statements will print their output in the workflow step results:

```bash
MESSAGE='Hello world'

# The message will now be available in the "Result > Logs" area in the workflow step
echo $MESSAGE
```

## Making an HTTP request

`curl` is already preinstalled in bash steps, we recommend using it for making HTTP requests in your code for sending or requesting data from APIs or webpages.

### Making a GET request

You can use `curl` to perform GET requests from websites or APIs directly.

```bash
# get your current public IP address from ifconfig.me
IP_ADDRESS=`curl --silent https://ifconfig.me/ip`
echo $IP_ADDRESS
```

::: tip
Use the `--silent` flag with `curl` to supress extra extra diagnostic information that `curl` produces when making requests.

This enables you to only worry about the body of the response so you can visualize it with tools like `echo` or `jq`. 
:::

### Making a POST request

`curl` handles making POSTs requests as well. The `-X` flag allow you to specify the HTTP method you'd like to use for an HTTP request.

The `-d` flag is for passing data in the POST request.

```bash
curl --silent -X POST https://postman-echo.com/post -d 'name=Bulbasaur&id=1'

# to store the API response in a variable, interpolate the response into a string and store it in variable
RESPONSE=`curl --silent -X POST https://postman-echo.com/post -d 'name=Bulbasaur&id=1'`

# now the response is stored as a variable
echo $RESPONSE
```

### Using API key authentication

Some APIs require you to use a secret API key to prove your access.

`curl` has an `-h` flag where you can pass your API key as a token.

Below is an example of searching Twitter for mentions of a specific Twitter handle. Twitter requires a valid API key passed in the header of the request.

```bash
curl --silent -X POST -h "authorization:Bearer $(<your api key here>)" https://api.twitter.com/2/users/@pipedream/mentions
```

## Sharing data between steps

A step can accept data from other steps in the same workflow, or pass data downstream to others.

This makes your steps even more powerful, you can compose new workflows and reuse steps.

### Using data from another step

In bash scripts, data from the initial workflow trigger and other steps are available in the `$PIPEDREAM_STEPS` environment variable.

In this example, we'll pretend this data is coming into our HTTP trigger via a POST request.

```json
{
  "id": 1,
  "name": 'Bulbasaur",
  "type": "plant"
}
```

In our bash script, we can access this data via the `$PIPEDREAM_STEPS` variable. Specifically, this data from the POST request into our workflow is available in the `trigger` object.

```bash
echo $PIPEDREAM_STEPS | jq .trigger.event

# results in { id: 1, name: "Bulbasaur", type: "plant" }
```

::: tip
The period (`.`) in front the `trigger.event` in the example is not a typo. This is to define the starting point for `jq` to start traversing down the JSON in the HTTP response.
:::

### Sending data downstream to other steps

To share data created, retrieved, transformed or manipulated by a step to others downstream, append it to the `$PIPEDREAM_EXPORTS` variable.

An example speaks a thousand words, so here's one passing data from an API to the bash step.

```bash
# Retrieve the data from an API and store it in a variable
DATA=`curl --silent https://pokeapi.co/api/v2/pokemon/charizard`

# Write data to $PIPEDREAM_EXPORTS to share with steps downstream
EXPORT="key:json=${DATA}"
echo $EXPORT >> $PIPEDREAM_EXPORTS
```

::: tip 
Don't worry, the special `key` string in the `EXPORT` will automatically reference the current step's name. 

This way you won't have collisions with multiple bash scripts exporting data. Accessing the data is based off of the step's name, no need to edit the `key` string to try and name it something unique.
:::

## File storage

### Writing a file to /tmp

### Reading a file from /tmp

### Listing all files in /tmp

### `/tmp` limitations