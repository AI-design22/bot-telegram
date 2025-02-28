const https = require('https');

// Chỉ theo dõi ví có số đuôi "095"
const walletAddress = "0xaa4cf4517f4a4757a0b28c8438bfaba7eaefa095";

// Địa chỉ hợp đồng BRIL token trên Polygon
const brilTokenAddress = "0x4f800ba0dff2980c5006c6816f7aa3de63ce8087";

// API key của PolygonScan
const blockchainAPIKey = "HN4VF9PTKDVRBH3FXKIRSUUP9KVGEM35YM";

// Token của bot Telegram
const botToken = "7634692596:AAHe4cS5PlOT_Q_E4-YEvDiuyqcGPg699Ow";

// Chat ID của bạn
const chatID = "1670866660";

// Hàm gửi yêu cầu HTTP
function sendHttpRequest(url, callback) {
  https.get(url, (res) => {
    let data = '';

    res.on('data', (chunk) => {
      data += chunk;
    });

    res.on('end', () => {
      try {
        callback(JSON.parse(data));
      } catch (err) {
        console.error("Lỗi khi xử lý JSON: ", err.message);
        callback(null);
      }
    });
  }).on("error", (err) => {
    console.log("Error: " + err.message);
  });
}

// Lấy số dư của token BRIL
async function getBrilBalance(address) {
  const url = `https://api.polygonscan.com/api?module=account&action=tokenbalance&contractaddress=${brilTokenAddress}&address=${address}&tag=latest&apikey=${blockchainAPIKey}`;

  return new Promise((resolve, reject) => {
    sendHttpRequest(url, (data) => {
      if (data && data.status === "1") {
        let balance = parseFloat(data.result) / 1e18;
        resolve(balance);
      } else {
        reject(`Lỗi khi lấy số dư cho ví ${address}`);
      }
    });
  });
}

// Gửi tin nhắn đến Telegram
function sendTelegramMessage(message) {
  const url = `https://api.telegram.org/bot${botToken}/sendMessage?chat_id=${chatID}&text=${encodeURIComponent(message)}`;

  sendHttpRequest(url, (data) => {
    if (data && data.ok) {
      console.log("Tin nhắn đã được gửi đến Telegram.");
    } else {
      console.log("Gửi tin nhắn thất bại. Lỗi: ", data?.description || "Không rõ.");
      if (data?.error_code) {
        console.log("Mã lỗi:", data.error_code);
      }
    }
  });
}

// Kiểm tra số dư và gửi thông báo
async function checkBalancesAndNotify() {
  try {
    const balance = await getBrilBalance(walletAddress);

    if (balance !== null) {
      const message = `💰 Số dư ví:\n\n🔹 Ví: ${walletAddress}\n🔹 Số dư: ${balance.toFixed(6)} BRIL`;
      sendTelegramMessage(message);
    }
  } catch (error) {
    console.error("Lỗi kiểm tra số dư: ", error);
  }
}

// Bắt đầu kiểm tra số dư và gửi thông báo
checkBalancesAndNotify();
