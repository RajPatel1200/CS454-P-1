# Project 1 – EC2 REST Service: Pounds → Kilograms

# Summary
This project provisions an AWS EC2 instance and deploy a small REST web service that converts pounds (lbs) to kilograms (kg).

# Learning Objectives
- Provision and secure a VM in the public cloud (AWS EC2).
- Expose a minimal REST API over HTTP.
- Use system services (systemd) to run a web service reliably.
- Apply basic DevOps practices: logging, reproducible setup instructions.
- Practice cloud cost hygiene and cleanup
# Required API Specification

- Implement an HTTP GET endpoint that converts pounds to kilograms:
  `GET /convert?lbs=<number>`

**Response: 200 OK, application/json**
```json
{
"lbs": <number>,
"kg": <number rounded to 3 decimals>,
"formula": "kg = lbs * 0.45359237"
}
```
# Errors:
- 400 Bad Request if query param missing or not a number.
- 422 Unprocessable Entity if value is negative or non-finite (NaN/Inf).

# Setup Steps 

1. Launch a t3.micro EC3 instance with Ubantu LTS, create a Key pair
2. Configure security group to Limit SSH to your IP
3. SSH into the Instance: Use your key pair to connect to the instance.
 ```bash
 -ssh -i /path/to/your/key.pem ubuntu@[Your Public IP]
```
4. Install Node.js and implement following service
```
mkdir -p ~/p1 && cd ~/p1
npm init -y
npm install express morgan
cat > server.js <<'EOF'
const express = require('express');
const morgan = require('morgan');
const app = express();
app.use(morgan('combined'));
app.get('/convert', (req, res) => {
const lbs = Number(req.query.lbs);
if (req.query.lbs === undefined || Number.isNaN(lbs)) {
return res.status(400).json({ error: 'Query param lbs is required and must be a number' });

}
if (!Number.isFinite(lbs) || lbs < 0) {
return res.status(422).json({ error: 'lbs must be a non-negative, finite number' });
}
const kg = Math.round(lbs * 0.45359237 * 1000) / 1000;
return res.json({ lbs, kg, formula: 'kg = lbs * 0.45359237' });
});
const port = process.env.PORT || 8080;
app.listen(port, () => console.log(`listening on ${port}`));
EOF
node server.js
Open your browser to http://<PUBLIC_IP>:8080/convert?lbs=150 or test with curl:
curl 'http://<PUBLIC_IP>:8080/convert?lbs=150'
```
5. Run as Service (systemd)
  ```bash
 # Create a systemd unit
sudo bash -c 'cat >/etc/systemd/system/p1.service <<"UNIT"\n[Unit]\nDescription=CS454 Project 1 service\nAfter=network.target\n\n[Service]\nUser=ec2-user\nWorkingDirectory=/home/ec2-user/p1\nExecStart=/usr/bin/node /home/ec2-user/p1/server.js\nRestart=always\nEnvironment=PORT=8080\n\n[Install]\nWantedBy=multi-user.target\nUNIT'
sudo systemctl daemon-reload
sudo systemctl enable --now p1
sudo systemctl status p1 --no-pager
```
6. Test cases and Output
<img width="1920" height="1200" alt="Screenshot 2025-09-24 170811" src="https://github.com/user-attachments/assets/c507ed0c-91d9-498c-b6c8-650ef51bd594" />

7. Cleanup & Cost notes
   - EC2 instance has been terminated and deleted to avoid further costs
   - Associated key pair has been deleted as well


