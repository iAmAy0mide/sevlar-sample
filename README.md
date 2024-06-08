# sevlar-sample

To replicate; 
  <li>npm install</li>
  <li>Create a .env file in the 'src' folder </li>
  <li>Add the following:
    <li>PORT=portnumber</li>
    <li>JWT_SECRET=secret</li>
    <li>MONGO_URI=mongo_uri</li>
  </li>

<br>
This repository is a small part of another large, private project that I am currently working on. The project aims so solve the hassle people face when dealing with many banks by providing modern interface to connect all their banks, view, make and manage transacions from each bank accounts. Not only can user create budgets but they can also manage expenses - All of these tailored with robust authentication and authorizations. This sample project is created to replicate a small functionality of the larger project.
  As nodejs developers, we always strive to maintain the principle of 'separation of concerns'. One way to implement this could be by making sure that 'ModelA' doesn't directly modify or call a function from 'ModelB'. And often times when implementing this, we get this error which force us to restructure our pre (we should, obviously).

<pre>[ERR_HTTP_HEADERS_SENT]: Cannot set headers after they are sent to the client </pre>

Though this error can occur due to many reasons. But it's obvious that we're setting the 'headers' twice as the indicated by the error message. I try to solve this error while maintaining the principle of 'separation of concern', by creating a utility function that returns a boolean (true or false) if the headers has already been set somewhere. I then only need to check this returned value and act accordingly.

<pre> 
  import Transaction from "../models/transaction.mongo.js";
  const checkTransactionOwnership = async (userId, transactionId, req, res) => {
    const transaction = await Transaction.findById(transactionId);
    let method;
    method = req.method.toLowerCase();
    if (method === "put") {
        method = "update";
    }

    if (!transaction) {
        res.status(404).json({ message: "Transaction not found."});
        return false;
    }

    if (transaction.userId.toString() !== userId) {
        res.status(403).json({ message: `Unauthorized - You can only ${method} your own transactions` });
        return false;
    }

    return true;
    }
    ...
    export {
        checkTransactionOwnership,
    }
</pre>

The actual implementation of this is used here: 

<pre>
  ...
    const getTransaction = asyncHandler(async (req, res) => {
    const { transactionId } = req.params;
    const userID = req.user?._id.toString();

    if (!transactionId) {
        return res.status(400).json({ message: "Transaction id is required." });
    }

    if (typeof transactionId !== "string") {
        return res.status(400).json({ message: "Transaction id is invalid." });
    }

    const checked = await checkTransactionOwnership(userID, transactionId, req, res);

    if (!checked) {
        return;
    }

    const transaction = await fetchATransaction(transactionId);

    return res.status(200).json(transaction);
    });
  ...
    export {
        getTransaction
    }
</pre>
