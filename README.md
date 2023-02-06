# Chat AI by Jay Kang 
Hello! Welcome to a AI system that was built using JavaScript, NodeJs/Vite, OpenAI, and HTML/CSS. This webservice is inspired by ChatGPT, and uses the OpenAI's machine learning model. 

## Getting Started
**Prerequisites**:
* This project was built using NodeJS + Vite v4.0.0

**Installation**:
* NodeJS Download here: (https://nodejs.org/en/)

**Dependencies:**
* `npm create vite@latest client --template vanilla` (Creates a vanilla JS repo)
* `cd client`
* `npm install`
* These same steps are applied for a `server` folder: Replace `client` with `server
* **Make sure you install server outside of the client folder!**
* Place Assets folder into `client`: https://drive.google.com/file/d/1RhtfgrDaO7zoHIJgTUOZKYGdzTFJpe7V/view
* style.css: Override current css with pre-coded css (https://gist.github.com/adrianhajdin/2059ca74452a18d1560aac9499f58900)
* `npm run dev` to run the client

## Setup
**index.html**:
`<link rel="icon" type="image/svg+xml" href="favicon.ico" />`
* Replace the href with "favicon.ico"
Link Internal Styles Sheet: `<link rel="stylesheet" href="style.css" />`
```
  <body>
    <div id="app">
      <div id="chat_container"></div>

      <form>
        <textarea name="prompt" rows="1" cols="1" placeholder="Ask codex..."></textarea>
        <button type="submit"><img src="assets/send.svg" alt="send" />
      </form>
    </div>

    <script type="module" src="script.js"></script>
  </body>
```
* Make sure this is included in your body: This will create the form for user to type, and change the `main.js` file with `script.js`



## CLIENT (script.js)
**Here we have two classes: `class Item` and `class Receipt`**
* These classes each represent a table with certain attributes, where the fields are respective of the receipt requirements and properties (from API.yml file)
* Each class also contains a `@staticmethod` which takes in an object as a dictionary , which then initializes the object's properties using the extracted information
  * Returns a new instance of object Receipt or Item with the extracted information
  * `@staticmethod` is a Python decorator that is used for methods not dependent on the state of the object
* Note imported python libraries which help specify object's field attributes
```
class Receipt:
    retailer: str
    purchaseDate: date
    purchaseTime: time
    items: List[Item]
    total: decimal
    id: str = field(init=False)   # These variables are not automatically initialized by the dataclass
    points: int = field(init=False)
```
* `id` and `point` attributes contain `field(init=False)` due to the instantiation of App, it will look for these attributes from the incoming JSON body, although they have not been created
* Must make boolean False in order to bypass attributes

## Receipts
**receipts.py** <br/>
This is our blueprint, which register a set of operations at a certain url prefix. Here we are creating our API endpoints for our `POST` request and `GET` request.
```
@receipts.route('/process', methods=['POST'])
def process_receipt():
    if not request.is_json:
        return 'The receipt is invalid', 400

    jsonStr = request.get_json()
    receipt = Receipt.from_dict(jsonStr)

    receipt.id = str(uuid4())

    receipt.points = Points.calculate(receipt)

    Repo.store_receipt(receipt)

    return jsonify(
        id=receipt.id
    ), 200

```
In our `POST` method, this is what we are doing:
* Validating if incoming JSON Body is in json format
* Parse the incoming JSON to instantiate an instance of `Receipt` class
* Generate UUID using imported library `(uuid4())`
* Calculate points using our `rules.py` and `Points` class
* Storing information in our `Repo.py` IN MEMORY SOLUTION
* Return UUID as JSON str 

```
@receipts.route('/<id>/points', methods=['GET'])
def get_points(id):
    receipt = Repo.get_receipt(id)

    # validate receipt exists
    if receipt is None:
        return 'No receipt found for that id', 404

    # return the points value
    return jsonify(
        points=receipt.points
    ), 200
```
In our `GET` request, this is what we are doing:
* Looking up what we stored in the repo with generated uuid
* Validating if receipt exists
* Returning the total points value in a JSON str

## Rules
**rules.py** <br/>
We calculate rules by creating an Object for each rule implemented. Each object contains a static method with a common signature <br/>
* Allows for additional rules to be easily implemented later 

**points.py** <br/>
This page contains the `class Points`. Using the `def calculate()` method, will iterate over the rules list and return the total point amount. 

## Repo: In-memory Solution
Our in-memory solution uses a class `Repo` which stores the receipt information into a dictionary. This class contains two static methods: 
```
    @staticmethod
    def store_receipt(receipt : Receipt):
        Repo.receipt_dict[receipt.id] = receipt
        return

    @staticmethod
    def get_receipt(id : int):
        return Repo.receipt_dict.get(id)
```
* `store_receipt()` is used in the `POST` request to store the object receipt in repo
* `get_points(id)` is used in the `GET` request to get the calculated points

## Things to improve upon:
* The `POST` request validates if the receipt is JSON, but it does not validate whether the JSON contains all the required fields necessary <br/>
  * To improve this, would create statement to iterate through attributes, and generate error if requirements of receipt JSON are not met
* The in-memory solution can be implemented for a database in the future, such as `SQL-Alchemy`

## Design Question:
**Where and when to calculate points** <br/>
Right now, the points are calculated and stored in the `POST` `/process` call. This means the points will be calculated only once and cached. Multiple "points" calls are efficient with this method. However, if the rules change in the future, all receipts must be re-processed. <br />
If we were to calculate on demand in the `/points` call, this would mean any rule changes would be automatically applied and not re-processed. However, there would be more processing if the rules and receipts remain the same, meaning the same calculation is done multiiple times.

## Contact
Jay Kang - (kang.jay.87@gmail.com)
Project Link: (https://github.com/kangjay88/receiptprocessor.git)

## Acknowledgements
* Flask documentation: (https://flask.palletsprojects.com/en/1.1.x/)
* JSON Body -> Object: (https://jsonformatter.org/json-to-python)
* README.md file inspired by: (https://github.com/othneildrew/Best-README-Template)
