const fs = require('fs');
const path = require('path');

const FILE = path.join('/tmp', 'fch-reservations.json');
let reservations = [];

function load() {
  try {
    if (fs.existsSync(FILE)) {
      reservations = JSON.parse(fs.readFileSync(FILE, 'utf8'));
    }
  } catch (e) {
    reservations = [];
  }
}

function save() {
  try {
    fs.writeFileSync(FILE, JSON.stringify(reservations));
  } catch (e) {}
}

module.exports = async (req, res) => {
  res.setHeader('Access-Control-Allow-Origin', '*');
  res.setHeader('Access-Control-Allow-Methods', 'GET, POST, PATCH, DELETE, OPTIONS');
  res.setHeader('Access-Control-Allow-Headers', 'Content-Type');
  res.setHeader('Cache-Control', 'no-store');

  if (req.method === 'OPTIONS') {
    res.status(204).end();
    return;
  }

  load();

  if (req.method === 'GET') {
    res.status(200).json({ ok: true, reservations });
    return;
  }

  if (req.method === 'POST') {
    const body = typeof req.body === 'string' ? JSON.parse(req.body) : req.body;
    const reservation = {
      id: 'RES_' + Date.now(),
      customer_name: body.customer_name || 'Guest',
      customer_phone: body.customer_phone || '',
      party_size: body.party_size || 2,
      reservation_date: body.reservation_date || new Date().toISOString().split('T')[0],
      reservation_time: body.reservation_time || '19:00',
      table_number: body.table_number || null,
      special_requests: body.special_requests || '',
      status: body.status || 'pending',
      created_at: new Date().toISOString()
    };
    reservations.push(reservation);
    if (reservations.length > 100) reservations = reservations.slice(-100);
    save();

    res.status(200).json({ ok: true, reservation });
    return;
  }

  res.status(405).json({ ok: false, error: 'method_not_allowed' });
};
