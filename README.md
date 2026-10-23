# BTC-Cor-cd
rm -rf manglacoin && mkdir manglacoin && cd manglacoin
cat > requirements.txt << 'EOF'
Flask
gunicorn
EOF
cat > app.py << 'PYEOF'
from flask import Flask, jsonify, render_template, request
import hashlib, time, json, os, uuid
app = Flask(__name__)
BLOCK_REWARD=50
CHAIN_FILE='chain.json'
WALLETS_FILE='wallets.json'
def load_chain():
    if os.path.exists(CHAIN_FILE):
        import json; f=open(CHAIN_FILE); return json.load(f)
    genesis={'index':0,'timestamp':time.time(),'transactions':[],'previous_hash':'0'*64,'nonce':0,'hash':hashlib.sha256(b'genesis').hexdigest()}
    return [genesis]
def load_wallets():
    if os.path.exists(WALLETS_FILE):
        import json; f=open(WALLETS_FILE); return json.load(f)
    return {}
chain=load_chain()
wallets=load_wallets()
def save_data():
    import json
    json.dump(chain,open(CHAIN_FILE,'w')); json.dump(wallets,open(WALLETS_FILE,'w'))
def get_wallet(name):
    for addr,data in wallets.items():
        if data['name'].lower()==name.lower(): return addr,data
    addr='1Mangla'+uuid.uuid4().hex[:25]; priv=uuid.uuid4().hex+uuid.uuid4().hex
    wallets[addr]={'addr':addr,'balance':0,'name':name,'priv':priv}; return addr,wallets[addr]
@app.route('/')
def home(): return render_template('index.html')
@app.route('/chain')
def get_chain(): return jsonify(chain)
@app.route('/wallets')
def get_wallets(): return jsonify(wallets)
@app.route('/mine/<name>')
def mine(name):
    addr,wallet=get_wallet(name); last=chain[-1]
    new={'index':len(chain),'timestamp':time.time(),'transactions':[{'to':addr,'amount':50}],'previous_hash':last['hash'],'nonce':int(time.time())}
    new['hash']=hashlib.sha256(json.dumps(new,sort_keys=True).encode()).hexdigest()
    chain.append(new); wallets[addr]['balance']+=50; save_data()
    return jsonify({'message':f'Block {new["index"]} mined!','balance':wallets[addr]['balance']})
if __name__=='__main__':
    port=int(os.environ.get('PORT',5000)); app.run(host='0.0.0.0',port=port)
PYEOF
mkdir -p templates
cat > templates/index.html << 'HTM'
<!doctype html><title>ManglaCoin - Permanent</title>
<h1 style="text-align:center">ManglaCoin - Permanent Blockchain</h1>
<p style="text-align:center">From Mangla, Azad Kashmir</p>
<div style="text-align:center"><input id="n" placeholder="Your Name"><button onclick="mine()">Mine 50 MGC</button></div>
<pre id="o"></pre>
<script>
async function mine(){
 let name=document.getElementById('n').value||'Ali';
 let r=await fetch('/mine/'+name); let j=await r.json();
 document.getElementById('o').innerText=JSON.stringify(j,null,2);
}
</script>
HTM
git init
git add .
git config --global user.email "a@b.com"
git config --global user.name "Ali"
git commit -m "launch"
git branch -M main
git remote add origin https://github.com/bigol6568-pixel/manglacoin.git
