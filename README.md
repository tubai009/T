// POST /send-otp
app.post('/send-otp', async (req, res) => {
  const { phone } = req.body;
  const otp = Math.floor(100000 + Math.random() * 900000);
  await OTPModel.create({ phone, otp });
  // Use service like Twilio, Firebase to send OTP
  res.send({ message: 'OTP sent' });
});

// POST /verify-otp
app.post('/verify-otp', async (req, res) => {
  const { phone, otp } = req.body;
  const record = await OTPModel.findOne({ phone, otp });
  if (record) {
    const token = jwt.sign({ phone }, 'SECRET_KEY');
    res.send({ token });
  } else {
    res.status(400).send({ error: 'Invalid OTP' });
  }
});
// POST /deposit
app.post('/deposit', async (req, res) => {
  const { amount, userId } = req.body;
  const instance = new Razorpay({ key_id, key_secret });
  const order = await instance.orders.create({
    amount: amount * 100,
    currency: "INR",
    receipt: `user-${userId}-${Date.now()}`
  });
  res.send({ orderId: order.id });
});

// POST /withdraw (bank account/UPI)
app.post('/withdraw', async (req, res) => {
  const { userId, amount, upiId } = req.body;
  if (amount < 100 || amount > 10000) return res.status(400).send({ error: 'Invalid amount' });
  const user = await User.findById(userId);
  if (user.balance < amount) return res.status(400).send({ error: 'Insufficient balance' });
  // Transfer via Razorpay Payout API
  user.balance -= amount;
  await user.save();
  res.send({ success: true });
});
// After successful deposit
user.coins += depositAmount; // ₹100 = 100 coins

// On first signup
user.coins += Math.floor(1 + Math.random() * 10); // ₹1 to ₹10
// POST /spin
app.post('/spin', async (req, res) => {
  const { userId, bet } = req.body;
  const user = await User.findById(userId);
  if (user.coins < bet) return res.status(400).send({ error: 'Insufficient coins' });

  const outcome = Math.random() < 0.2 ? 'win' : 'lose';
  const result = outcome === 'win' ? bet * 2 : 0;
  user.coins = user.coins - bet + result;
  await user.save();
  res.send({ result: outcome, coins: user.coins });
});
// GET /profile
app.get('/profile/:id', async (req, res) => {
  const user = await User.findById(req.params.id);
  res.send({
    phone: user.phone,
    coins: user.coins,
    username: user.username
  });
});
[
  { name: "GEMS FORTUNE", minBet: 1, maxBet: 10000 },
  { name: "DIAMOND 777", minBet: 10, maxBet: 5000 },
  { name: "POWER OF THE KRAKEN III", minBet: 20, maxBet: 2000 }
]
// POST /chat
app.post('/chat', async (req, res) => {
  const userMessage = req.body.message;
  const response = await dialogflowClient.detectIntent({ session, query: userMessage });
  res.send({ reply: response.fulfillmentText });
});
