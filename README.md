/* 🚀 CHATGPT CLONE – FULL PRODUCTION-LIKE DEMO */

import { useState, useEffect } from "react";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";

/* 🔐 DEFAULT ADMIN */
const DEFAULT_USERS: Record<string, { password: string; role: string }> = {
  taha5: { password: "281216", role: "admin" },
};

/* 🌐 SAFE ENV ACCESS */
const API_URL =
  typeof import.meta !== "undefined" &&
  import.meta.env &&
  import.meta.env.VITE_API_URL
    ? import.meta.env.VITE_API_URL
    : "http://localhost:3001";

const PERSONAS = {
  default: "Sen yardımcı bir yapay zekasın.",
  coder: "Sen kıdemli bir yazılımcısın.",
  teacher: "Sen bir öğretmensin, sade anlat.",
};

type Message = { role: "user" | "ai"; text: string };
type Chat = { owner: string; title: string; messages: Message[] };

export default function App() {
  const [users, setUsers] = useState<Record<string, any>>(() => {
    try {
      return JSON.parse(localStorage.getItem("users") || "") || DEFAULT_USERS;
    } catch {
      return DEFAULT_USERS;
    }
  });

  const [username, setUsername] = useState("");
  const [password, setPassword] = useState("");
  const [showPass, setShowPass] = useState(false);
  const [user, setUser] = useState<string | null>(null);
  const [mode, setMode] = useState<"login" | "register">("login");

  const [persona, setPersona] = useState<keyof typeof PERSONAS>("default");
  const [question, setQuestion] = useState("");
  const [typing, setTyping] = useState(false);
  const [search, setSearch] = useState("");

  const [currentChatId, setCurrentChatId] = useState<string | null>(null);
  const [chats, setChats] = useState<Record<string, Chat>>(() => {
    try {
      return JSON.parse(localStorage.getItem("chats") || "") || {};
    } catch {
      return {};
    }
  });

  useEffect(() => {
    localStorage.setItem("users", JSON.stringify(users));
    localStorage.setItem("chats", JSON.stringify(chats));
  }, [users, chats]);

  /* 🔐 LOGIN / REGISTER */
  if (!user) {
    return (
      <div className="h-screen flex items-center justify-center bg-black text-white">
        <div className="w-80 space-y-3 p-6 rounded-xl border">
          <h2 className="text-xl font-bold text-center">
            {mode === "login" ? "Giriş Yap" : "Kayıt Ol"}
          </h2>

          <Input
            placeholder="Kullanıcı adı"
            value={username}
            onChange={(e) => setUsername(e.target.value)}
          />

          <div className="relative">
            <Input
              type={showPass ? "text" : "password"}
              placeholder="Şifre"
              value={password}
              onChange={(e) => setPassword(e.target.value)}
            />
            <button
              className="absolute right-2 top-2 text-xs"
              onClick={() => setShowPass(!showPass)}
            >
              {showPass ? "🙈" : "👁️"}
            </button>
          </div>

          {mode === "login" ? (
            <Button
              className="w-full"
              onClick={() => {
                if (!users[username] || users[username].password !== password) {
                  alert("Hatalı giriş");
                  return;
                }
                setUser(username);
              }}
            >
              Giriş
            </Button>
          ) : (
            <Button
              className="w-full"
              onClick={() => {
                if (password.length < 6) {
                  alert("Şifre en az 6 karakter");
                  return;
                }
                if (users[username]) {
                  alert("Kullanıcı var");
                  return;
                }
                setUsers({ ...users, [username]: { password, role: "user" } });
                setMode("login");
                alert("Kayıt başarılı");
              }}
            >
              Kaydol
            </Button>
          )}

          <button
            className="text-sm underline"
            onClick={() =>
              setMode(mode === "login" ? "register" : "login")
            }
          >
            {mode === "login" ? "Kayıt Ol" : "Giriş Yap"}
          </button>
        </div>
      </div>
    );
  }

  /* ➕ YENİ SOHBET */
  const newChat = () => {
    const id = Date.now().toString();
    setChats({
      ...chats,
      [id]: { owner: user, title: "Yeni Sohbet", messages: [] },
    });
    setCurrentChatId(id);
  };

  /* 🤖 AI SOR */
  const ask = async () => {
    if (!question || !currentChatId) return;

    const updated: Message[] = [
      ...chats[currentChatId].messages,
      { role: "user", text: question },
    ];

    const title =
      chats[currentChatId].title === "Yeni Sohbet"
        ? question.slice(0, 30)
        : chats[currentChatId].title;

    setChats({
      ...chats,
      [currentChatId]: { ...chats[currentChatId], title, messages: updated },
    });

    setQuestion("");
    setTyping(true);

    try {
      const res = await fetch(`${API_URL}/ask`, {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify({ messages: updated, persona }),
      });
      const data = await res.json();
      updated.push({ role: "ai", text: data.answer || "Cevap yok" });
    } catch {
      updated.push({
        role: "ai",
        text: "(Demo) AI cevabı – backend yok",
      });
    }

    setChats({
      ...chats,
      [currentChatId]: { ...chats[currentChatId], messages: updated },
    });
    setTyping(false);
  };

  return (
    <div className="flex h-screen bg-black text-white">
      <div className="w-64 border-r p-2 hidden md:block">
        <Button className="w-full" onClick={newChat}>
          + Yeni Sohbet
        </Button>
        <Input
          placeholder="Ara"
          value={search}
          onChange={(e) => setSearch(e.target.value)}
        />

        {Object.entries(chats)
          .filter(([, c]) => c.owner === user)
          .filter(([, c]) =>
            c.title.toLowerCase().includes(search.toLowerCase())
          )
          .map(([id, c]) => (
            <div
              key={id}
              onClick={() => setCurrentChatId(id)}
              className="p-2 cursor-pointer hover:bg-gray-800"
            >
              {c.title}
            </div>
          ))}
      </div>

      <div className="flex-1 flex flex-col">
        <div className="p-2 border-b flex justify-between items-center">
          <select
            value={persona}
            onChange={(e) =>
              setPersona(e.target.value as keyof typeof PERSONAS)
            }
          >
            <option value="default">Varsayılan</option>
            <option value="coder">Yazılımcı</option>
            <option value="teacher">Öğretmen</option>
          </select>
          <button className="text-sm underline" onClick={() => setUser(null)}>
            Çıkış
          </button>
        </div>

        <div className="flex-1 overflow-y-auto p-4 space-y-3">
          {currentChatId &&
            chats[currentChatId].messages.map((m, i) => (
              <div
                key={i}
                className={m.role === "user" ? "text-right" : "text-left"}
              >
                <span className="inline-block p-3 bg-gray-700 rounded-2xl max-w-xl">
                  {m.text}
                </span>
              </div>
            ))}
          {typing && (
            <div className="italic opacity-70 animate-pulse">
              AI yazıyor…
            </div>
          )}
        </div>

        <div className="p-3 flex gap-2 border-t">
          <Input
            value={question}
            onChange={(e) => setQuestion(e.target.value)}
          />
          <Button onClick={ask}>Gönder</Button>
        </div>
      </div>
    </div>
  );
}
