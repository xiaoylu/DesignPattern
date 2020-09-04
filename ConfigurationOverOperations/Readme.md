Configuration over Operations
===

This topic might be too big.

* Web Browser: UI rendering (drawing components on screen) is separated from the HTML DOMs. Web server sends a "model" file (HTML) to browser.
* Tensorflow: users specify a model (#layers, structure: lstm vs. transformer etc.). The execution is separated from the model definition.

We create a model/configuration input for a program, which makes it more scalable and maintainable.
