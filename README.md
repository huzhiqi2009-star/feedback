```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>匿名意见箱</title>
    <script src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
    <script src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
    <script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
    <script src="https://cdn.tailwindcss.com"></script>
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-app.js";
        import { getFirestore, collection, addDoc, query, onSnapshot, orderBy, serverTimestamp } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-firestore.js";
        import { getAuth, signInAnonymously, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/10.8.0/firebase-auth.js";

        // --- 重点：请把你在 Firebase 获得的配置粘贴在这里 ---
        const firebaseConfig = {
            apiKey: "在此处填入API_KEY",
            authDomain: "在此处填入PROJECT_ID.firebaseapp.com",
            projectId: "在此处填入PROJECT_ID",
            storageBucket: "在此处填入PROJECT_ID.appspot.com",
            messagingSenderId: "在此处填入SENDER_ID",
            appId: "在此处填入APP_ID"
        };

        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);
        window.fb = { auth, db, collection, addDoc, query, onSnapshot, orderBy, serverTimestamp, signInAnonymously, onAuthStateChanged };
    </script>
</head>
<body class="bg-gray-50">
    <div id="root"></div>
    <script type="text/babel">
        const { useState, useEffect } = React;
        const { auth, db, collection, addDoc, query, onSnapshot, orderBy, serverTimestamp, signInAnonymously, onAuthStateChanged } = window.fb;

        function App() {
            const [user, setUser] = useState(null);
            const [msg, setMsg] = useState('');
            const [status, setStatus] = useState('');
            const [admin, setAdmin] = useState(false);
            const [list, setList] = useState([]);
            const [pw, setPw] = useState('');

            useEffect(() => {
                signInAnonymously(auth).catch(e => setStatus('配置错误，请检查 Firebase 参数'));
                onAuthStateChanged(auth, setUser);
            }, []);

            useEffect(() => {
                if (admin && user) {
                    const q = query(collection(db, "messages"), orderBy("timestamp", "desc"));
                    return onSnapshot(q, (s) => setList(s.docs.map(d => ({id: d.id, ...d.data()}))));
                }
            }, [admin, user]);

            const send = async () => {
                if (!msg.trim()) return;
                setStatus('发送中...');
                try {
                    await addDoc(collection(db, "messages"), { content: msg, timestamp: serverTimestamp() });
                    setMsg(''); setStatus('✅ 已成功匿名提交！');
                    setTimeout(() => setStatus(''), 3000);
                } catch (e) { setStatus('❌ 提交失败，数据库规则未开启？'); }
            };

            return (
                <div className="max-w-md mx-auto p-6 pt-12">
                    <h1 className="text-2xl font-bold text-center mb-2">匿名意见箱</h1>
                    <p className="text-gray-400 text-center text-sm mb-8">在这里留下你的想法，我看得到。</p>
                    
                    {!admin ? (
                        <div className="space-y-4">
                            <textarea className="w-full h-40 p-4 border rounded-2xl focus:ring-2 focus:ring-black outline-none shadow-sm" 
                                placeholder="输入内容 (1000字以内)" value={msg} onChange={e=>setMsg(e.target.value)} maxLength={1000}/>
                            <button onClick={send} className="w-full bg-black text-white py-3 rounded-xl font-bold">匿名发送</button>
                            {status && <p className="text-center text-sm font-medium text-blue-600">{status}</p>}
                        </div>
                    ) : (
                        <div className="space-y-4">
                            <button onClick={()=>setAdmin(false)} className="text-xs text-gray-400">返回</button>
                            {list.map(i => (
                                <div key={i.id} className="bg-white p-4 rounded-xl border shadow-sm">
                                    <p className="text-gray-800">{i.content}</p>
                                    <p className="text-[10px] text-gray-300 mt-2">{i.timestamp?.toDate().toLocaleString()}</p>
                                </div>
                            ))}
                        </div>
                    )}
                    
                    <div className="mt-20 opacity-10 flex justify-center gap-2">
                        <input type="password" placeholder="密码" className="border text-xs w-20 px-1" value={pw} onChange={e=>setPw(e.target.value)}/>
                        <button onClick={() => pw === "admin123" && setAdmin(true)} className="text-xs">查看</button>
                    </div>
                </div>
            );
        }
        ReactDOM.createRoot(document.getElementById('root')).render(<App />);
    </script>
</body>
</html>

```
