import { useState, useEffect, useRef } from "react";
import {
  Heart, MessageCircle, Share2, Bell, Image as ImageIcon, Lock, Unlock,
  Search, User, Settings as SettingsIcon, Calendar, FileText, ShieldCheck, X,
  Camera, Send, Ban, CheckCircle2, XCircle, Home as HomeIcon,
  Users, Clock, Flag, ChevronLeft, BarChart2, Crown, MessageSquare,
  LifeBuoy, ScrollText, Mail, KeyRound, BadgeCheck, LogOut, PlusCircle,
  Rocket, Gift, MoreVertical, Trash2, EyeOff, Mic, Copy, Upload,
  MessagesSquare, Radio
} from "lucide-react";

/* ---------- Design tokens ----------
Palette: green-deep #0B6E3E / green-ink #063C22 / paper #F7F5EE / gold #C79A2E / charcoal #1E211D / line #DCD7C7
Type: Fraunces (display) / Inter (body). Signature: broom-notch edge on Chairman card.

WIRED TO REAL SUPABASE (auth + profiles/roles): Register and Login call the real
Supabase Auth API. Whether Admin Panel / Chairman Panel shows is decided by the
profiles.is_admin / profiles.is_chairman columns for that account — set manually
in Supabase, not chosen at registration.

STILL LOCAL PROTOTYPE (no database table wired yet — needs its own Supabase table
before it's truly "live"): posts/comments, Messages, Notifications, Meeting,
Poll, Support tickets, Verification requests + receipts, Banners. These work in
this session but reset on reload until we build their tables next.
------------------------------------- */

const SUPABASE_URL = "https://yudkakryupxxefaejbsn.supabase.co";
const SUPABASE_ANON_KEY = "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJpc3MiOiJzdXBhYmFzZSIsInJlZiI6Inl1ZGtha3J5dXB4eGVmYWVqYnNuIiwicm9sZSI6ImFub24iLCJpYXQiOjE3ODcwMDIxNTAsImV4cCI6MjEwMjU3ODE1MH0.63Bh9f5HXH1deRNN-BQsJnQwPKqpWU-t8U69Q5VLH78";

const BANK = { name: "Umar Idris", bank: "GTBank", account: "1033297039" };
const PRIVACY_TEXT = "Brazilteam only collects the information members choose to share: name, phone, email, and posts made on the platform. This information is used to run the community and is never sold. Only Admin and the Chairman can view member contact details, and only when needed to respond to a request or a report.";
const TERMS_TEXT = "By using Brazilteam, members agree to communicate with respect. Abusive language or content that insults another member or the party is not allowed. Any member can report a post; reported posts are reviewed in the Admin Panel. Repeated violations can lead to a suspended account. The Chairman's messages are official and may not be edited or deleted by anyone else.";

async function sbSignUp(email, password, fullName) {
  const res = await fetch(`${SUPABASE_URL}/auth/v1/signup`, {
    method: "POST",
    headers: { "Content-Type": "application/json", apikey: SUPABASE_ANON_KEY },
    body: JSON.stringify({ email, password, data: { full_name: fullName } }),
  });
  const data = await res.json();
  if (!res.ok) throw new Error(data.msg || data.error_description || "Registration failed.");
  return data;
}
async function sbSignIn(email, password) {
  const res = await fetch(`${SUPABASE_URL}/auth/v1/token?grant_type=password`, {
    method: "POST",
    headers: { "Content-Type": "application/json", apikey: SUPABASE_ANON_KEY },
    body: JSON.stringify({ email, password }),
  });
  const data = await res.json();
  if (!res.ok) throw new Error(data.error_description || data.msg || "Login failed.");
  return data;
}
async function sbGet(path, token) {
  const res = await fetch(`${SUPABASE_URL}/rest/v1/${path}`, {
    headers: { apikey: SUPABASE_ANON_KEY, Authorization: `Bearer ${token}` },
  });
  if (!res.ok) return null;
  return res.json();
}
async function sbInsert(path, token, body) {
  const res = await fetch(`${SUPABASE_URL}/rest/v1/${path}`, {
    method: "POST",
    headers: { "Content-Type": "application/json", apikey: SUPABASE_ANON_KEY, Authorization: `Bearer ${token}`, Prefer: "return=representation,resolution=merge-duplicates" },
    body: JSON.stringify(body),
  });
  if (!res.ok) return null;
  return res.json();
}
async function sbPatch(path, token, body) {
  const res = await fetch(`${SUPABASE_URL}/rest/v1/${path}`, {
    method: "PATCH",
    headers: { "Content-Type": "application/json", apikey: SUPABASE_ANON_KEY, Authorization: `Bearer ${token}`, Prefer: "return=representation" },
    body: JSON.stringify(body),
  });
  if (!res.ok) return null;
  return res.json();
}

function Avatar({ letter, size = 10, gold = false }) {
  return (
    <div className="flex items-center justify-center rounded-full font-semibold text-white shrink-0"
      style={{ width: `${size * 4}px`, height: `${size * 4}px`, background: gold ? "linear-gradient(135deg,#C79A2E,#8f6d1c)" : "linear-gradient(135deg,#0B6E3E,#063C22)", fontFamily: "Fraunces, serif" }}>
      {letter}
    </div>
  );
}
function NavButton({ active, icon, label, onClick }) {
  return (
    <button onClick={onClick} className="relative flex-1 flex flex-col items-center gap-0.5 py-2">
      <div style={{ color: active ? "#0B6E3E" : "#8a8c86" }}>{icon}</div>
      <span className="text-[10px] font-medium" style={{ color: active ? "#0B6E3E" : "#8a8c86" }}>{label}</span>
      {active && <div className="absolute -top-2 left-1/2 -translate-x-1/2 w-8 h-0.5 rounded-full" style={{ background: "#0B6E3E" }} />}
    </button>
  );
}
function SectionTitle({ children, icon }) { return <p className="text-sm font-semibold mb-2 flex items-center gap-1.5">{icon}{children}</p>; }
function InfoModal({ title, icon, onClose, children }) {
  return (
    <div className="fixed inset-0 z-50 bg-black/40 flex items-end sm:items-center justify-center">
      <div className="bg-white w-full sm:max-w-md sm:rounded-xl rounded-t-xl max-h-[85vh] overflow-y-auto p-5">
        <div className="flex items-center justify-between mb-3"><p className="brz-display text-lg font-semibold flex items-center gap-2">{icon}{title}</p><button onClick={onClose}><X size={18} /></button></div>
        {children}
      </div>
    </div>
  );
}
function ImagePicker({ images, setImages, small }) {
  const ref = useRef(null);
  const onFiles = (e) => {
    const files = Array.from(e.target.files || []);
    files.forEach((f) => {
      const reader = new FileReader();
      reader.onload = () => setImages((imgs) => [...imgs, reader.result]);
      reader.readAsDataURL(f);
    });
    e.target.value = "";
  };
  return (
    <div>
      <input ref={ref} type="file" accept="image/*" multiple hidden onChange={onFiles} />
      <button onClick={() => ref.current?.click()} className="flex items-center gap-1 text-xs text-neutral-600"><ImageIcon size={small ? 13 : 15} /> Photo</button>
      {images.length > 0 && (
        <div className="flex gap-1.5 flex-wrap mt-2">
          {images.map((src, i) => (
            <div key={i} className="relative">
              <img src={src} className="w-14 h-14 object-cover rounded-lg" />
              <button onClick={() => setImages((imgs) => imgs.filter((_, idx) => idx !== i))} className="absolute -top-1.5 -right-1.5 bg-black/70 rounded-full w-4 h-4 flex items-center justify-center"><X size={10} className="text-white" /></button>
            </div>
          ))}
        </div>
      )}
    </div>
  );
}

function AuthScreen({ onAuth }) {
  const [mode, setMode] = useState("register");
  const [fullName, setFullName] = useState("");
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");
  const [busy, setBusy] = useState(false);
  const [error, setError] = useState("");
  const [notice, setNotice] = useState("");

  const submit = async () => {
    setError(""); setNotice(""); setBusy(true);
    try {
      if (mode === "register") {
        const data = await sbSignUp(email, password, fullName);
        if (data.access_token) {
          onAuth(data.access_token, data.user, fullName);
        } else {
          setNotice("Account created. Please check your email to confirm it, then log in.");
          setMode("login");
        }
      } else {
        const data = await sbSignIn(email, password);
        onAuth(data.access_token, data.user, null);
      }
    } catch (e) {
      setError(e.message);
    }
    setBusy(false);
  };

  return (
    <div className="min-h-screen flex flex-col items-center justify-center px-6 relative overflow-hidden" style={{ background: "linear-gradient(160deg,#0B6E3E 0%,#063C22 100%)", fontFamily: "Inter, sans-serif" }}>
      <style>{`@import url('https://fonts.googleapis.com/css2?family=Fraunces:wght@600;700&family=Inter:wght@400;500;600;700&display=swap'); .brz-display{font-family:'Fraunces',serif;}`}</style>
      <div className="absolute inset-0 opacity-[0.06]" style={{ backgroundImage: "repeating-linear-gradient(45deg, white 0, white 2px, transparent 2px, transparent 26px)" }} />
      <div className="absolute -bottom-16 -right-16 w-56 h-56 rounded-full" style={{ background: "radial-gradient(circle,#C79A2E33,transparent 70%)" }} />

      <div className="relative w-14 h-14 rounded-2xl flex items-center justify-center mb-3" style={{ background: "#C79A2E" }}><span className="brz-display text-2xl font-bold text-white">B</span></div>
      <p className="relative brz-display text-white text-2xl font-semibold mb-6">Brazilteam</p>

      <div className="relative w-full max-w-sm bg-white rounded-2xl p-5">
        <p className="brz-display text-lg font-semibold mb-3">{mode === "register" ? "Create your account" : "Welcome back"}</p>
        {error && <p className="text-xs mb-2 px-3 py-2 rounded-lg" style={{ background: "#FBEAEA", color: "#C0392B" }}>{error}</p>}
        {notice && <p className="text-xs mb-2 px-3 py-2 rounded-lg" style={{ background: "#EAF3EE", color: "#0B6E3E" }}>{notice}</p>}
        <div className="space-y-2.5">
          {mode === "register" && (
            <div className="flex items-center gap-2 border rounded-lg px-3 py-2" style={{ borderColor: "#DCD7C7" }}><User size={15} className="text-neutral-400" /><input value={fullName} onChange={(e) => setFullName(e.target.value)} placeholder="Full name" className="flex-1 outline-none text-sm" /></div>
          )}
          <div className="flex items-center gap-2 border rounded-lg px-3 py-2" style={{ borderColor: "#DCD7C7" }}><Mail size={15} className="text-neutral-400" /><input value={email} onChange={(e) => setEmail(e.target.value)} placeholder="Email" className="flex-1 outline-none text-sm" /></div>
          <div className="flex items-center gap-2 border rounded-lg px-3 py-2" style={{ borderColor: "#DCD7C7" }}><KeyRound size={15} className="text-neutral-400" /><input value={password} onChange={(e) => setPassword(e.target.value)} type="password" placeholder="Password" className="flex-1 outline-none text-sm" /></div>
        </div>
        <button disabled={busy} onClick={submit} className="w-full mt-4 text-sm font-medium py-2.5 rounded-lg text-white disabled:opacity-60" style={{ background: "#0B6E3E" }}>
          {busy ? "Please wait..." : mode === "register" ? "Register" : "Log In"}
        </button>
        <p className="text-center text-xs text-neutral-500 mt-3">
          {mode === "register" ? (
            <>Already have an account? <button onClick={() => { setMode("login"); setError(""); setNotice(""); }} className="font-medium" style={{ color: "#0B6E3E" }}>Log in</button></>
          ) : (
            <>Don't have an account? <button onClick={() => { setMode("register"); setError(""); setNotice(""); }} className="font-medium" style={{ color: "#0B6E3E" }}>Register</button></>
          )}
        </p>
      </div>
    </div>
  );
}

export default function App() {
  const [session, setSession] = useState(null);
  const [profile, setProfile] = useState(null);
  const [users, setUsers] = useState([]);

  const [view, setView] = useState("main");
  const [tab, setTab] = useState("home");
  const [posts, setPosts] = useState([]);
  const [chPosts, setChPosts] = useState([]);
  const [newPostText, setNewPostText] = useState("");
  const [newPostImages, setNewPostImages] = useState([]);
  const [newChairmanText, setNewChairmanText] = useState("");
  const [newChairmanImages, setNewChairmanImages] = useState([]);
  const [openMenuPostId, setOpenMenuPostId] = useState(null);

  const [adminOpen, setAdminOpen] = useState(false);
  const [chairmanPanelOpen, setChairmanPanelOpen] = useState(false);
  const [infoModal, setInfoModal] = useState(null);

  const [banners, setBanners] = useState([{ id: 1, type: "text", text: "Chairman Brazil — Leader, Visionary", image: null }]);
  const [bannerIdx, setBannerIdx] = useState(0);
  const [bannerDraft, setBannerDraft] = useState("");
  const [bannerImgDraft, setBannerImgDraft] = useState(null);

  useEffect(() => {
    if (banners.length < 2) return;
    const t = setInterval(() => setBannerIdx((i) => (i + 1) % banners.length), 5000);
    return () => clearInterval(t);
  }, [banners.length]);

  const [homeSearch, setHomeSearch] = useState("");
  const [viewedUser, setViewedUser] = useState(null);
  const [followingIds, setFollowingIds] = useState([]);

  const [formAccessGranted, setFormAccessGranted] = useState(false);
  const [formRequestMsg, setFormRequestMsg] = useState("");
  const [formAccessRequests, setFormAccessRequests] = useState([]);
  const [formSubmissions, setFormSubmissions] = useState([]);
  const [formDraft, setFormDraft] = useState({ title: "", details: "" });

  const [meeting, setMeeting] = useState({ title: "", date: "", time: "", live: false, commentsLocked: false, comments: [] });
  const [meetingDraft, setMeetingDraft] = useState({ date: "", time: "", title: "" });
  const [meetingCommentText, setMeetingCommentText] = useState("");
  const [meetingCommentImages, setMeetingCommentImages] = useState([]);
  const [poll, setPoll] = useState(null);
  const [pollDraft, setPollDraft] = useState({ question: "", candidates: ["", ""] });
  const [votedPoll, setVotedPoll] = useState(false);

  const [violationQueue, setViolationQueue] = useState([]);
  const [adminSearch, setAdminSearch] = useState("");
  const [chairmanSearch, setChairmanSearch] = useState("");
  const [giftTarget, setGiftTarget] = useState(null);
  const [giftText, setGiftText] = useState("");

  const [supportTickets, setSupportTickets] = useState([]);
  const [supportDraft, setSupportDraft] = useState("");
  const [replyDrafts, setReplyDrafts] = useState({});

  const [chatLog, setChatLog] = useState([]);
  const [adminChatDraft, setAdminChatDraft] = useState("");
  const [chairmanChatDraft, setChairmanChatDraft] = useState("");

  const [verificationRequests, setVerificationRequests] = useState([]);
  const [freeVerifyEmail, setFreeVerifyEmail] = useState("");
  const [receiptFile, setReceiptFile] = useState(null);
  const [senderName, setSenderName] = useState("");
  const [followerEdits, setFollowerEdits] = useState({});
  const [notifications, setNotifications] = useState([]);
  const [noticeText, setNoticeText] = useState("");
  const [noticeTarget, setNoticeTarget] = useState("everyone");
  const [chairmanNoticeText, setChairmanNoticeText] = useState("");
  const [passwords, setPasswords] = useState({ current: "", next: "" });
  const [messagePrivacy, setMessagePrivacy] = useState("everyone");

  const [conversations, setConversations] = useState([]);
  const [activeThread, setActiveThread] = useState(null);
  const [threadDraft, setThreadDraft] = useState("");

  const role = profile ? (profile.is_chairman ? "chairman" : profile.is_admin ? "admin" : "user") : "user";
  const verified = !!profile?.verified;
  const myName = profile?.full_name || profile?.email || "You";

  const loadProfile = async (token, userObj, fallbackName) => {
    let rows = await sbGet(`profiles?id=eq.${userObj.id}&select=*`, token);
    if (!rows || rows.length === 0) {
      rows = await sbInsert("profiles", token, { id: userObj.id, email: userObj.email, full_name: fallbackName || userObj.email, is_admin: false, is_chairman: false, followers: 0, following: 0, likes_received: 0, verified: false });
    }
    if (rows && rows[0]) setProfile(rows[0]);
    const all = await sbGet("profiles?select=*", token);
    if (all) setUsers(all);
  };

  const handleAuth = async (token, userObj, fallbackName) => {
    setSession({ token, user: userObj });
    await loadProfile(token, userObj, fallbackName);
  };

  const logout = () => { setSession(null); setProfile(null); setUsers([]); setTab("home"); setView("main"); };

  if (!session || !profile) {
    return <AuthScreen onAuth={handleAuth} />;
  }

  const toggleLike = (id) => setPosts((p) => p.map((post) => (post.id === id ? { ...post, liked: !post.liked, likes: post.liked ? post.likes - 1 : post.likes + 1 } : post)));
  const addPost = () => {
    if (!newPostText.trim() && newPostImages.length === 0) return;
    setPosts((p) => [{ id: Date.now(), name: myName, time: "just now", text: newPostText, images: newPostImages, likes: 0, comments: 0, shares: 0, liked: false, reported: false, hidden: false }, ...p]);
    setNewPostText(""); setNewPostImages([]);
  };
  const deletePost = (id) => { setPosts((p) => p.filter((x) => x.id !== id)); setOpenMenuPostId(null); };
  const toggleHidePost = (id) => { setPosts((p) => p.map((x) => (x.id === id ? { ...x, hidden: !x.hidden } : x))); setOpenMenuPostId(null); };
  const reportPost = (post) => { setPosts((p) => p.map((x) => (x.id === post.id ? { ...x, reported: true } : x))); setViolationQueue((v) => [...v, { id: post.id, name: post.name, text: post.text, reason: "Reported by a member" }]); };

  const addChairmanPost = () => {
    if (!newChairmanText.trim() && newChairmanImages.length === 0) return;
    setChPosts((p) => [{ id: Date.now(), time: "just now", text: newChairmanText, images: newChairmanImages, likes: 0, comments: 0, shares: 0, commentsLocked: false }, ...p]);
    setNewChairmanText(""); setNewChairmanImages([]);
  };
  const toggleFollow = (id) => setFollowingIds((f) => (f.includes(id) ? f.filter((x) => x !== id) : [...f, id]));

  const requestFormAccess = () => { if (!formRequestMsg.trim()) return; setFormAccessRequests((r) => [...r, { id: Date.now(), name: myName, message: formRequestMsg, granted: false }]); setFormRequestMsg(""); };
  const grantAccess = (id) => { setFormAccessRequests((r) => r.map((x) => (x.id === id ? { ...x, granted: true } : x))); if (formAccessRequests.find((x) => x.id === id)?.name === myName) setFormAccessGranted(true); };
  const submitForm = () => { if (!formDraft.title.trim()) return; setFormSubmissions((s) => [...s, { id: Date.now(), name: myName, title: formDraft.title, details: formDraft.details, status: "pending" }]); setFormDraft({ title: "", details: "" }); setFormAccessGranted(false); };
  const decideSubmission = (id, status) => setFormSubmissions((s) => s.map((x) => (x.id === id ? { ...x, status } : x)));
  const decideViolation = (id) => setViolationQueue((v) => v.filter((x) => x.id !== id));

  const vote = (name) => { if (votedPoll || !poll) return; setPoll((p) => ({ ...p, candidates: p.candidates.map((c) => (c.name === name ? { ...c, votes: c.votes + 1 } : c)) })); setVotedPoll(true); };
  const addPollField = () => setPollDraft((d) => ({ ...d, candidates: [...d.candidates, ""] }));
  const updatePollField = (i, val) => setPollDraft((d) => ({ ...d, candidates: d.candidates.map((c, idx) => (idx === i ? val : c)) }));
  const startPoll = () => {
    const names = pollDraft.candidates.map((c) => c.trim()).filter(Boolean);
    if (!pollDraft.question || names.length < 2) return;
    setPoll({ question: pollDraft.question, candidates: names.map((n) => ({ name: n, votes: 0 })) });
    setVotedPoll(false); setPollDraft({ question: "", candidates: ["", ""] });
  };

  const submitTicket = () => { if (!supportDraft.trim()) return; setSupportTickets((t) => [...t, { id: Date.now(), name: myName, message: supportDraft, reply: null }]); setSupportDraft(""); };
  const sendReply = (id) => { const text = replyDrafts[id]; if (!text?.trim()) return; setSupportTickets((t) => t.map((x) => (x.id === id ? { ...x, reply: text } : x))); setReplyDrafts((r) => ({ ...r, [id]: "" })); };

  const sendChat = (from) => { const text = from === "admin" ? adminChatDraft : chairmanChatDraft; if (!text.trim()) return; setChatLog((c) => [...c, { id: Date.now(), from, text, time: "just now" }]); if (from === "admin") setAdminChatDraft(""); else setChairmanChatDraft(""); };
  const sendGift = () => { if (!giftTarget || !giftText.trim()) return; openThreadWith(giftTarget.full_name || giftTarget.email, giftText, true); setGiftText(""); setGiftTarget(null); };

  const publishMeeting = () => setMeeting((m) => ({ ...m, date: meetingDraft.date || m.date, time: meetingDraft.time || m.time, title: meetingDraft.title || m.title || "Members' Meeting" }));
  const goLive = () => setMeeting((m) => ({ ...m, live: true, commentsLocked: true }));
  const endLive = () => setMeeting((m) => ({ ...m, live: false, commentsLocked: false }));
  const sendMeetingComment = () => {
    if (meeting.commentsLocked && role === "user") return;
    if (!meetingCommentText.trim() && meetingCommentImages.length === 0) return;
    setMeeting((m) => ({ ...m, comments: [...m.comments, { id: Date.now(), name: myName, text: meetingCommentText, images: meetingCommentImages }] }));
    setMeetingCommentText(""); setMeetingCommentImages([]);
  };

  const copyBank = () => { navigator.clipboard?.writeText(`${BANK.name} — ${BANK.bank} — ${BANK.account}`); };
  const onReceiptFile = (e) => { const f = e.target.files?.[0]; if (!f) return; const reader = new FileReader(); reader.onload = () => setReceiptFile(reader.result); reader.readAsDataURL(f); };
  const submitVerification = () => {
    if (!receiptFile || !senderName.trim()) return;
    setVerificationRequests((r) => [...r.filter((x) => x.email !== profile.email), { id: Date.now(), name: myName, email: profile.email, senderName, receipt: receiptFile, status: "pending" }]);
    setReceiptFile(null); setSenderName("");
  };
  const approveVerification = async (req) => {
    setVerificationRequests((r) => r.map((x) => (x.id === req.id ? { ...x, status: "approved" } : x)));
    const target = users.find((u) => u.email === req.email);
    if (target) {
      const updated = await sbPatch(`profiles?id=eq.${target.id}`, session.token, { verified: true });
      if (updated) setUsers((us) => us.map((u) => (u.id === target.id ? updated[0] : u)));
      if (target.id === profile.id) setProfile((p) => ({ ...p, verified: true }));
    }
  };
  const giveFreeVerification = async () => {
    const target = users.find((u) => u.email === freeVerifyEmail.trim());
    if (!target) return;
    const updated = await sbPatch(`profiles?id=eq.${target.id}`, session.token, { verified: true });
    if (updated) setUsers((us) => us.map((u) => (u.id === target.id ? updated[0] : u)));
    if (target.id === profile.id) setProfile((p) => ({ ...p, verified: true }));
    setFreeVerifyEmail("");
  };

  const saveFollowerCount = async (u) => {
    const val = parseInt(followerEdits[u.id], 10);
    if (isNaN(val)) return;
    const before = u.followers || 0;
    if (Math.floor(val / 10) > Math.floor(before / 10)) {
      setNotifications((n) => [{ id: Date.now(), type: "milestone", text: `🎉 Congratulations ${u.full_name || u.email}, you reached ${val} followers!`, time: "just now", read: false }, ...n]);
    }
    const updated = await sbPatch(`profiles?id=eq.${u.id}`, session.token, { followers: val });
    if (updated) setUsers((us) => us.map((x) => (x.id === u.id ? updated[0] : x)));
  };

  const sendNotice = () => { if (!noticeText.trim()) return; const label = noticeTarget === "everyone" ? "Everyone" : noticeTarget; setNotifications((n) => [{ id: Date.now(), type: "notice", text: `Admin (to ${label}): "${noticeText}"`, time: "just now", read: false }, ...n]); setNoticeText(""); };
  const sendChairmanNotice = () => { if (!chairmanNoticeText.trim()) return; setNotifications((n) => [{ id: Date.now(), type: "notice", text: `Chairman Brazil (to Everyone): "${chairmanNoticeText}"`, time: "just now", read: false }, ...n]); setChairmanNoticeText(""); };
  const markRead = (id) => setNotifications((n) => n.map((x) => (x.id === id ? { ...x, read: true } : x)));

  const canMessage = (target) => {
    if (target.messagePrivacy === "locked") return false;
    if (target.messagePrivacy === "friends") return followingIds.includes(target.id) && (target.followingIds || []).includes(profile.id);
    return true;
  };
  const openThreadWith = (withName, firstText, fromChairman) => {
    setConversations((cs) => {
      const existing = cs.find((c) => c.withName === withName);
      const msg = { from: fromChairman ? "chairman" : "them", text: firstText, time: "just now" };
      if (existing) return cs.map((c) => (c.withName === withName ? { ...c, messages: [...c.messages, msg] } : c));
      return [...cs, { id: Date.now(), withName, messages: [msg] }];
    });
  };
  const sendThreadMessage = () => {
    if (!threadDraft.trim() || !activeThread) return;
    setConversations((cs) => cs.map((c) => (c.id === activeThread.id ? { ...c, messages: [...c.messages, { from: "me", text: threadDraft, time: "just now" }] } : c)));
    setThreadDraft("");
  };

  const totalVotes = poll ? poll.candidates.reduce((a, c) => a + c.votes, 0) || 1 : 1;
  const filteredUsers = users.filter((u) => (u.full_name || "").toLowerCase().includes(homeSearch.toLowerCase()) || (u.email || "").toLowerCase().includes(homeSearch.toLowerCase()));
  const adminFilteredUsers = users.filter((u) => (u.full_name || "").toLowerCase().includes(adminSearch.toLowerCase()) || (u.email || "").toLowerCase().includes(adminSearch.toLowerCase()));
  const chairmanFilteredUsers = users.filter((u) => (u.full_name || "").toLowerCase().includes(chairmanSearch.toLowerCase()) || (u.email || "").toLowerCase().includes(chairmanSearch.toLowerCase()));
  const unreadCount = notifications.filter((n) => !n.read).length;

  const navItems = [
    { id: "home", label: "Home", icon: <HomeIcon size={19} /> },
    { id: "chairman", label: "Mr. Chairman", icon: <ShieldCheck size={19} /> },
    { id: "form", label: "Form Request", icon: <FileText size={19} /> },
    { id: "meeting", label: "Meeting", icon: <Calendar size={19} /> },
    { id: "profile", label: "Profile", icon: <User size={19} /> },
  ];

  const currentBanner = banners[bannerIdx] || banners[0];

  return (
    <div className="min-h-screen w-full flex flex-col" style={{ background: "#F7F5EE", color: "#1E211D", fontFamily: "Inter, sans-serif" }}>
      <style>{`
        @import url('https://fonts.googleapis.com/css2?family=Fraunces:wght@500;600;700&family=Inter:wght@400;500;600;700&display=swap');
        .brz-display { font-family: 'Fraunces', serif; }
        .broom-notch { clip-path: polygon(0 0, 100% 0, 100% 100%, 14px 100%, 0 calc(100% - 14px)); }
      `}</style>

      {view === "notifications" && (
        <div className="fixed inset-0 z-50 bg-white flex flex-col">
          <header className="flex items-center gap-3 px-4 py-3 border-b" style={{ borderColor: "#DCD7C7" }}>
            <button onClick={() => setView("main")}><ChevronLeft size={20} /></button>
            <p className="brz-display text-lg font-semibold">Notifications</p>
          </header>
          <div className="flex-1 overflow-y-auto p-4 space-y-2">
            {notifications.length === 0 && <p className="text-sm text-neutral-400 text-center mt-10">No notifications yet.</p>}
            {notifications.map((n) => (
              <button key={n.id} onClick={() => markRead(n.id)} className="w-full text-left rounded-lg border p-3" style={{ borderColor: "#DCD7C7", background: n.read ? "white" : "#EAF3EE" }}>
                <p className="text-sm">{n.text}</p>
                <p className="text-xs text-neutral-400 mt-1">{n.time}</p>
              </button>
            ))}
          </div>
        </div>
      )}

      {view === "messages" && (
        <div className="fixed inset-0 z-50 bg-white flex flex-col">
          <header className="flex items-center gap-3 px-4 py-3 border-b" style={{ borderColor: "#DCD7C7" }}>
            <button onClick={() => setView("main")}><ChevronLeft size={20} /></button>
            <p className="brz-display text-lg font-semibold flex items-center gap-2"><MessagesSquare size={18} /> Messages</p>
          </header>
          <div className="flex-1 overflow-y-auto">
            {conversations.length === 0 && <p className="text-sm text-neutral-400 text-center mt-10">No conversations yet.</p>}
            {conversations.map((c) => (
              <button key={c.id} onClick={() => { setActiveThread(c); setView("thread"); }} className="w-full flex items-center gap-3 px-4 py-3 border-b text-left" style={{ borderColor: "#F0EEE4" }}>
                <Avatar letter={c.withName[0]} />
                <div className="flex-1 min-w-0">
                  <p className="text-sm font-semibold">{c.withName}</p>
                  <p className="text-xs text-neutral-500 truncate">{c.messages[c.messages.length - 1]?.text}</p>
                </div>
              </button>
            ))}
          </div>
        </div>
      )}

      {view === "thread" && activeThread && (
        <div className="fixed inset-0 z-50 bg-white flex flex-col">
          <header className="flex items-center gap-3 px-4 py-3 border-b" style={{ borderColor: "#DCD7C7" }}>
            <button onClick={() => setView("messages")}><ChevronLeft size={20} /></button>
            <Avatar letter={activeThread.withName[0]} size={8} />
            <p className="font-semibold text-sm">{activeThread.withName}</p>
          </header>
          <div className="flex-1 overflow-y-auto p-4 space-y-2">
            {activeThread.messages.map((m, i) => (
              <div key={i} className={`max-w-[75%] px-3 py-2 rounded-2xl text-sm ${m.from === "me" ? "ml-auto text-white" : ""}`} style={m.from === "me" ? { background: "#0B6E3E" } : { background: "#F0EEE4" }}>
                {m.text}
              </div>
            ))}
          </div>
          <div className="p-3 border-t flex gap-2" style={{ borderColor: "#DCD7C7" }}>
            <input value={threadDraft} onChange={(e) => setThreadDraft(e.target.value)} placeholder="Message..." className="flex-1 border rounded-full px-3 py-2 text-sm outline-none" style={{ borderColor: "#DCD7C7" }} onKeyDown={(e) => e.key === "Enter" && sendThreadMessage()} />
            <button onClick={sendThreadMessage} className="w-9 h-9 rounded-full flex items-center justify-center text-white" style={{ background: "#0B6E3E" }}><Send size={15} /></button>
          </div>
        </div>
      )}

      {view === "main" && (<>
      <header className="sticky top-0 z-30 flex items-center justify-between px-4 py-2.5 border-b" style={{ background: "#0B6E3E", borderColor: "#063C22" }}>
        <div className="flex items-center gap-2">
          <div className="w-8 h-8 rounded-md flex items-center justify-center" style={{ background: "#C79A2E" }}><span className="brz-display text-sm font-bold text-white">B</span></div>
          <span className="brz-display text-white text-lg font-semibold tracking-tight">Brazilteam</span>
        </div>
        <div className="flex items-center gap-2">
          <button onClick={() => setView("notifications")} className="relative w-9 h-9 rounded-full flex items-center justify-center" style={{ background: "rgba(255,255,255,0.12)" }}>
            <Bell size={17} className="text-white" />
            {unreadCount > 0 && <span className="absolute -top-0.5 -right-0.5 w-2.5 h-2.5 rounded-full" style={{ background: "#C79A2E" }} />}
          </button>
          <button onClick={() => setTab("profile")}><Avatar letter={myName[0]} size={8} /></button>
        </div>
      </header>

      <main className="flex-1 max-w-2xl w-full mx-auto px-4 py-4 space-y-4 pb-24">
        {tab === "home" && !viewedUser && (
          <>
            <div className="relative rounded-xl overflow-hidden" style={{ height: 190 }}>
              {currentBanner?.image ? (
                <img src={currentBanner.image} className="absolute inset-0 w-full h-full object-cover" />
              ) : (
                <div className="absolute inset-0" style={{ background: "linear-gradient(135deg,#0B6E3E 0%,#063C22 55%,#0B6E3E 100%)" }} />
              )}
              <div className="absolute inset-0 bg-black/20" />
              <div className="absolute inset-0 flex flex-col justify-end p-4">
                <p className="brz-display text-white text-2xl font-semibold">{currentBanner?.text}</p>
                <p className="text-white/80 text-xs mt-0.5">Party Chairman · Brazilteam</p>
              </div>
              {banners.length > 1 && <div className="absolute bottom-2 right-3 flex gap-1">{banners.map((_, i) => <span key={i} className="w-1.5 h-1.5 rounded-full" style={{ background: i === bannerIdx ? "#C79A2E" : "rgba(255,255,255,0.5)" }} />)}</div>}
            </div>

            <div className="relative">
              <div className="flex items-center gap-2 bg-white rounded-full px-3 py-2.5 border" style={{ borderColor: "#DCD7C7" }}>
                <Search size={16} className="text-neutral-500" />
                <input value={homeSearch} onChange={(e) => setHomeSearch(e.target.value)} placeholder="Search a member to view their profile..." className="flex-1 outline-none text-sm" />
              </div>
              {homeSearch && (
                <div className="absolute z-20 mt-1 w-full bg-white rounded-lg border shadow-lg overflow-hidden" style={{ borderColor: "#DCD7C7" }}>
                  {filteredUsers.map((u) => (
                    <button key={u.id} onClick={() => { setViewedUser(u); setHomeSearch(""); }} className="w-full flex items-center gap-2 px-3 py-2 text-left hover:bg-neutral-50">
                      <Avatar letter={(u.full_name || u.email)[0]} size={7} />
                      <div><p className="text-sm font-medium flex items-center gap-1">{u.full_name || u.email}{u.verified && <BadgeCheck size={13} style={{ color: "#2E7CD6" }} />}</p><p className="text-xs text-neutral-500">{u.email}</p></div>
                    </button>
                  ))}
                  {filteredUsers.length === 0 && <p className="px-3 py-2 text-xs text-neutral-500">No member found.</p>}
                </div>
              )}
            </div>

            <div className="rounded-xl border p-3" style={{ borderColor: "#DCD7C7", background: "white" }}>
              <div className="flex gap-2">
                <Avatar letter={myName[0]} size={8} />
                <input value={newPostText} onChange={(e) => setNewPostText(e.target.value)} placeholder="What's on your mind?" className="flex-1 rounded-full px-3 text-sm outline-none border" style={{ borderColor: "#DCD7C7" }} />
              </div>
              <div className="flex items-center justify-between mt-2 pt-2 border-t" style={{ borderColor: "#DCD7C7" }}>
                <ImagePicker images={newPostImages} setImages={setNewPostImages} />
                <button onClick={addPost} className="flex items-center gap-1 text-xs font-medium px-3 py-1.5 rounded-full text-white" style={{ background: "#0B6E3E" }}><Send size={13} /> Post</button>
              </div>
            </div>

            {posts.filter((p) => !p.hidden || p.name === myName).map((post) => {
              const author = users.find((u) => (u.full_name || u.email) === post.name);
              const isFollowing = author ? followingIds.includes(author.id) : true;
              const isMine = post.name === myName;
              return (
                <div key={post.id} className="rounded-xl border p-3 relative" style={{ borderColor: "#DCD7C7", background: post.hidden ? "#FAFAF7" : "white" }}>
                  <div className="flex items-center gap-2 mb-2">
                    <Avatar letter={post.name[0]} />
                    <div>
                      <p className="text-sm font-semibold flex items-center gap-1">{post.name}{isMine && verified && <BadgeCheck size={13} style={{ color: "#2E7CD6" }} />}</p>
                      <p className="text-xs text-neutral-500">{post.time}{post.hidden ? " · Hidden" : ""}</p>
                    </div>
                    {isMine && verified && <span className="ml-auto text-[10px] font-medium px-2 py-0.5 rounded-full flex items-center gap-1" style={{ background: "#EAF3EE", color: "#0B6E3E" }}><Rocket size={10} /> Boosted</span>}
                    {!isMine && <span className="ml-auto text-[10px] px-2 py-0.5 rounded-full" style={isFollowing ? { background: "#EAF3EE", color: "#0B6E3E" } : { background: "#F0EEE4", color: "#8a8c86" }}>{isFollowing ? "Following" : "Suggested"}</span>}
                    {isMine && (
                      <div className="relative">
                        <button onClick={() => setOpenMenuPostId(openMenuPostId === post.id ? null : post.id)} className="ml-1 text-neutral-400"><MoreVertical size={16} /></button>
                        {openMenuPostId === post.id && (
                          <div className="absolute right-0 mt-1 w-32 bg-white rounded-lg border shadow-lg z-10 text-xs" style={{ borderColor: "#DCD7C7" }}>
                            <button onClick={() => toggleHidePost(post.id)} className="w-full flex items-center gap-1.5 px-3 py-2 hover:bg-neutral-50"><EyeOff size={12} /> {post.hidden ? "Unhide" : "Hide"}</button>
                            <button onClick={() => deletePost(post.id)} className="w-full flex items-center gap-1.5 px-3 py-2 hover:bg-neutral-50" style={{ color: "#C0392B" }}><Trash2 size={12} /> Delete</button>
                          </div>
                        )}
                      </div>
                    )}
                  </div>
                  {post.text && <p className="text-sm leading-relaxed">{post.text}</p>}
                  {post.images?.length > 0 && <div className="grid grid-cols-2 gap-1.5 mt-2">{post.images.map((src, i) => <img key={i} src={src} className="rounded-lg w-full h-32 object-cover" />)}</div>}
                  <div className="flex items-center gap-5 mt-3 text-xs text-neutral-600">
                    <button onClick={() => toggleLike(post.id)} className="flex items-center gap-1" style={post.liked ? { color: "#C0392B" } : {}}><Heart size={14} fill={post.liked ? "#C0392B" : "none"} /> {post.likes}</button>
                    <span className="flex items-center gap-1"><MessageCircle size={14} /> {post.comments}</span>
                    <span className="flex items-center gap-1"><Share2 size={14} /> {post.shares}</span>
                    {!post.reported && !isMine && <button onClick={() => reportPost(post)} className="ml-auto flex items-center gap-1 text-neutral-400 hover:text-red-500"><Flag size={13} /> Report</button>}
                  </div>
                </div>
              );
            })}
            {posts.length === 0 && <p className="text-center text-sm text-neutral-400 py-8">No posts yet — be the first to share something.</p>}
          </>
        )}

        {tab === "home" && viewedUser && (
          <div className="space-y-4">
            <button onClick={() => setViewedUser(null)} className="flex items-center gap-1 text-sm font-medium" style={{ color: "#0B6E3E" }}><ChevronLeft size={16} /> Back to Home</button>
            <div className="rounded-xl border overflow-hidden" style={{ borderColor: "#DCD7C7", background: "white" }}>
              <div style={{ height: 80, background: "linear-gradient(135deg,#0B6E3E,#C79A2E)" }} />
              <div className="px-4 pb-4">
                <div className="-mt-8 mb-2"><Avatar letter={(viewedUser.full_name || viewedUser.email)[0]} size={16} /></div>
                <p className="brz-display text-lg font-semibold flex items-center gap-1">{viewedUser.full_name || viewedUser.email}{viewedUser.verified && <BadgeCheck size={16} style={{ color: "#2E7CD6" }} />}</p>
                <p className="text-xs text-neutral-500 mb-3">{viewedUser.email}</p>
                <div className="flex gap-4 text-sm mb-3">
                  <span><strong>{viewedUser.followers || 0}</strong> Followers</span>
                  <span><strong>{viewedUser.following || 0}</strong> Following</span>
                  <span><strong>{viewedUser.likes_received || 0}</strong> Likes</span>
                </div>
                <div className="flex gap-2">
                  <button onClick={() => toggleFollow(viewedUser.id)} className="text-xs font-medium px-4 py-1.5 rounded-full" style={followingIds.includes(viewedUser.id) ? { background: "#F0EEE4", color: "#063C22" } : { background: "#0B6E3E", color: "white" }}>
                    {followingIds.includes(viewedUser.id) ? "Following ✓" : "Follow"}
                  </button>
                  {canMessage(viewedUser) && viewedUser.id !== profile.id && (
                    <button onClick={() => { const name = viewedUser.full_name || viewedUser.email; openThreadWith(name, "👋 Hello!"); setActiveThread(conversations.find((c) => c.withName === name) || { id: Date.now(), withName: name, messages: [] }); setView("thread"); }} className="text-xs font-medium px-4 py-1.5 rounded-full border" style={{ borderColor: "#0B6E3E", color: "#0B6E3E" }}>Message</button>
                  )}
                </div>
              </div>
            </div>
          </div>
        )}

        {tab === "chairman" && (
          <div className="space-y-4">
            <div className="flex items-center gap-3 rounded-xl p-4" style={{ background: "linear-gradient(135deg,#0B6E3E,#063C22)" }}>
              <Avatar letter="B" gold size={12} />
              <div><p className="brz-display text-white text-lg font-semibold">Chairman Brazil</p><p className="text-white/70 text-xs">Only the Chairman posts here — everyone can read</p></div>
            </div>
            {role === "chairman" && (
              <div className="rounded-xl border p-3" style={{ borderColor: "#C79A2E", background: "#FFFDF6" }}>
                <textarea value={newChairmanText} onChange={(e) => setNewChairmanText(e.target.value)} placeholder="Write a message to all members..." rows={3} className="w-full text-sm outline-none resize-none" />
                <div className="flex items-center justify-between mt-2"><ImagePicker images={newChairmanImages} setImages={setNewChairmanImages} /><button onClick={addChairmanPost} className="flex items-center gap-1 text-xs font-medium px-3 py-1.5 rounded-full text-white" style={{ background: "#C79A2E" }}><Send size={13} /> Publish</button></div>
              </div>
            )}
            {chPosts.map((cp) => (
              <div key={cp.id} className="broom-notch rounded-xl p-4 border-2" style={{ borderColor: "#C79A2E", background: "#FFFDF6" }}>
                <div className="flex items-center gap-2 mb-2">
                  <Avatar letter="B" gold />
                  <div><p className="text-sm font-semibold flex items-center gap-1">Chairman Brazil <ShieldCheck size={14} style={{ color: "#C79A2E" }} /></p><p className="text-xs text-neutral-500">{cp.time}</p></div>
                </div>
                {cp.text && <p className="text-sm leading-relaxed">{cp.text}</p>}
                {cp.images?.length > 0 && <div className="grid grid-cols-2 gap-1.5 mt-2">{cp.images.map((src, i) => <img key={i} src={src} className="rounded-lg w-full h-32 object-cover" />)}</div>}
                <div className="flex items-center gap-5 mt-3 text-xs text-neutral-600">
                  <span className="flex items-center gap-1"><Heart size={14} /> {cp.likes}</span>
                  <span className="flex items-center gap-1"><MessageCircle size={14} /> {cp.comments}</span>
                  <span className="flex items-center gap-1"><Share2 size={14} /> {cp.shares}</span>
                  {role === "chairman" && <button onClick={() => setChPosts((arr) => arr.map((p) => (p.id === cp.id ? { ...p, commentsLocked: !p.commentsLocked } : p)))} className="ml-auto flex items-center gap-1 font-medium" style={{ color: "#0B6E3E" }}><Lock size={12} /> {cp.commentsLocked ? "Unlock comments" : "Lock comments"}</button>}
                </div>
              </div>
            ))}
            {chPosts.length === 0 && <p className="text-center text-sm text-neutral-400 py-8">The Chairman hasn't posted yet.</p>}
          </div>
        )}

        {tab === "form" && (
          <div className="rounded-xl border p-5" style={{ borderColor: "#DCD7C7", background: "white" }}>
            <p className="brz-display text-lg font-semibold mb-1">Form Request</p>
            {!formAccessGranted && (
              <>
                <p className="text-xs text-neutral-500 mb-4">This form is closed by default. Contact the Admin with your reason — once approved, the form opens for you here.</p>
                <div className="space-y-3">
                  <textarea value={formRequestMsg} onChange={(e) => setFormRequestMsg(e.target.value)} placeholder="Tell the Admin what you need and why..." rows={3} className="w-full border rounded-lg px-3 py-2 text-sm outline-none" style={{ borderColor: "#DCD7C7" }} />
                  <button onClick={requestFormAccess} className="flex items-center gap-1 text-sm font-medium px-4 py-2 rounded-lg text-white" style={{ background: "#0B6E3E" }}><MessageSquare size={14} /> Contact Admin to get Form</button>
                </div>
              </>
            )}
            {formAccessGranted && (
              <div className="space-y-3">
                <input value={formDraft.title} onChange={(e) => setFormDraft((d) => ({ ...d, title: e.target.value }))} placeholder="Request title" className="w-full border rounded-lg px-3 py-2 text-sm outline-none" style={{ borderColor: "#DCD7C7" }} />
                <textarea value={formDraft.details} onChange={(e) => setFormDraft((d) => ({ ...d, details: e.target.value }))} placeholder="Describe what you need..." rows={4} className="w-full border rounded-lg px-3 py-2 text-sm outline-none" style={{ borderColor: "#DCD7C7" }} />
                <button onClick={submitForm} className="text-sm font-medium px-4 py-2 rounded-lg text-white" style={{ background: "#0B6E3E" }}>Submit to Chairman</button>
              </div>
            )}
          </div>
        )}

        {tab === "meeting" && (
          <div className="space-y-4">
            <div className="rounded-xl border overflow-hidden" style={{ borderColor: "#DCD7C7", background: "white" }}>
              <div className="p-4" style={{ background: "linear-gradient(135deg,#C79A2E,#8f6d1c)" }}>
                <p className="brz-display text-white text-lg font-semibold">{meeting.title || "No meeting scheduled"}</p>
                {meeting.date && <p className="text-white/80 text-xs mt-1 flex items-center gap-1"><Clock size={12} /> {meeting.date}, {meeting.time}</p>}
                {meeting.live && <p className="text-white text-xs mt-1 flex items-center gap-1 font-semibold"><Radio size={12} /> LIVE now</p>}
              </div>
              <div className="p-4 space-y-3">
                {(role === "admin" || role === "chairman") && (
                  <div className="grid grid-cols-2 gap-2">
                    <input value={meetingDraft.date} onChange={(e) => setMeetingDraft((d) => ({ ...d, date: e.target.value }))} placeholder="Date" className="border rounded-lg px-2 py-1.5 text-xs outline-none" style={{ borderColor: "#DCD7C7" }} />
                    <input value={meetingDraft.time} onChange={(e) => setMeetingDraft((d) => ({ ...d, time: e.target.value }))} placeholder="Time" className="border rounded-lg px-2 py-1.5 text-xs outline-none" style={{ borderColor: "#DCD7C7" }} />
                    <input value={meetingDraft.title} onChange={(e) => setMeetingDraft((d) => ({ ...d, title: e.target.value }))} placeholder="Meeting title" className="col-span-2 border rounded-lg px-2 py-1.5 text-xs outline-none" style={{ borderColor: "#DCD7C7" }} />
                    <button onClick={publishMeeting} className="col-span-2 text-xs font-medium px-3 py-1.5 rounded-lg text-white" style={{ background: "#0B6E3E" }}>Publish Meeting Notice</button>
                    {!meeting.live ? <button onClick={goLive} className="col-span-2 text-xs font-medium px-3 py-1.5 rounded-lg text-white flex items-center justify-center gap-1" style={{ background: "#C0392B" }}><Radio size={13} /> Join as Host (go live & lock comments)</button>
                      : <button onClick={endLive} className="col-span-2 text-xs font-medium px-3 py-1.5 rounded-lg" style={{ background: "#F0EEE4" }}>End Host Session</button>}
                  </div>
                )}
                <p className="text-xs text-neutral-500 pt-1">Before a host joins, everyone can chat freely below. Once Admin or Chairman joins, comments lock until reopened.</p>
              </div>
            </div>

            <div className="rounded-xl border p-4" style={{ borderColor: "#DCD7C7", background: "white" }}>
              <SectionTitle icon={<MessageCircle size={15} style={{ color: "#0B6E3E" }} />}>Meeting Chat</SectionTitle>
              <div className="max-h-56 overflow-y-auto space-y-2 mb-2">
                {meeting.comments.map((c) => (
                  <div key={c.id} className="text-xs"><strong>{c.name}:</strong> {c.text} {c.images?.length > 0 && <span className="text-neutral-400">[photo attached]</span>}</div>
                ))}
                {meeting.comments.length === 0 && <p className="text-xs text-neutral-400">No messages yet.</p>}
              </div>
              {(!meeting.commentsLocked || role !== "user") ? (
                <div className="space-y-1.5">
                  <input value={meetingCommentText} onChange={(e) => setMeetingCommentText(e.target.value)} placeholder="Say something..." className="w-full border rounded-lg px-3 py-2 text-sm outline-none" style={{ borderColor: "#DCD7C7" }} />
                  <div className="flex items-center justify-between">
                    <div className="flex items-center gap-3">
                      <ImagePicker images={meetingCommentImages} setImages={setMeetingCommentImages} small />
                      <button className="flex items-center gap-1 text-xs text-neutral-400" title="Voice notes coming once storage is connected"><Mic size={13} /> Voice</button>
                    </div>
                    <button onClick={sendMeetingComment} className="text-xs font-medium px-3 py-1.5 rounded-full text-white" style={{ background: "#0B6E3E" }}>Send</button>
                  </div>
                </div>
              ) : <p className="text-xs text-neutral-400 flex items-center gap-1"><Lock size={12} /> Comments are locked while the host is live.</p>}
            </div>

            {poll && (
              <div className="rounded-xl border p-4" style={{ borderColor: "#DCD7C7", background: "white" }}>
                <SectionTitle icon={<BarChart2 size={15} style={{ color: "#0B6E3E" }} />}>Live Poll</SectionTitle>
                <p className="text-sm mb-3">{poll.question}</p>
                <div className="space-y-2">
                  {poll.candidates.map((c) => {
                    const pct = Math.round((c.votes / totalVotes) * 100);
                    return (
                      <button key={c.name} disabled={votedPoll || role !== "user"} onClick={() => vote(c.name)} className="w-full text-left">
                        <div className="flex justify-between text-xs mb-1"><span className="font-medium">{c.name}</span><span>{pct}%</span></div>
                        <div className="w-full h-2 rounded-full bg-neutral-100 overflow-hidden"><div className="h-full" style={{ width: `${pct}%`, background: "#0B6E3E" }} /></div>
                      </button>
                    );
                  })}
                </div>
                {votedPoll && <p className="text-[11px] mt-2" style={{ color: "#0B6E3E" }}>✓ Your vote was recorded.</p>}
              </div>
            )}

            {role === "chairman" && (
              <div className="rounded-xl border p-4 space-y-2" style={{ borderColor: "#C79A2E", background: "#FFFDF6" }}>
                <SectionTitle icon={<Crown size={15} style={{ color: "#C79A2E" }} />}>Create Poll (add as many candidates as needed)</SectionTitle>
                <input value={pollDraft.question} onChange={(e) => setPollDraft((d) => ({ ...d, question: e.target.value }))} placeholder="Poll question" className="w-full border rounded-lg px-2 py-1.5 text-xs outline-none" style={{ borderColor: "#DCD7C7" }} />
                {pollDraft.candidates.map((c, i) => (
                  <input key={i} value={c} onChange={(e) => updatePollField(i, e.target.value)} placeholder={`Candidate ${String.fromCharCode(65 + i)}`} className="w-full border rounded-lg px-2 py-1.5 text-xs outline-none" style={{ borderColor: "#DCD7C7" }} />
                ))}
                <button onClick={addPollField} className="flex items-center gap-1 text-xs font-medium" style={{ color: "#0B6E3E" }}><PlusCircle size={13} /> Add another candidate</button>
                <button onClick={startPoll} className="text-xs font-medium px-3 py-1.5 rounded-lg text-white w-full" style={{ background: "#C79A2E" }}>Start Poll</button>
              </div>
            )}
          </div>
        )}

        {tab === "profile" && (
          <div className="space-y-4">
            <div className="rounded-xl border overflow-hidden" style={{ borderColor: "#DCD7C7", background: "white" }}>
              <div style={{ height: 90, background: "linear-gradient(135deg,#0B6E3E,#C79A2E)" }} />
              <div className="px-4 pb-4">
                <div className="-mt-8 mb-2"><Avatar letter={myName[0]} size={16} /></div>
                <div className="flex items-center gap-2">
                  <p className="brz-display text-lg font-semibold flex items-center gap-1">{myName} {verified && <BadgeCheck size={17} style={{ color: "#2E7CD6" }} />}</p>
                  <button onClick={() => setViewedUser({ ...profile, full_name: myName })} className="text-neutral-400" title="View my full profile"><User size={15} /></button>
                </div>
                <p className="text-xs text-neutral-500 mb-3">{role === "chairman" ? "Chairman · verified" : role === "admin" ? "Admin" : "Member"} · Brazilteam</p>
                <div className="flex gap-4 text-sm mb-3">
                  <span><strong>{profile.followers || 0}</strong> Followers</span>
                  <span><strong>{profile.following || 0}</strong> Following</span>
                  <span><strong>{profile.likes_received || 0}</strong> Likes</span>
                </div>

                {role === "admin" && <button onClick={() => setAdminOpen(true)} className="mt-1 w-full flex items-center justify-center gap-2 text-sm font-medium px-4 py-2.5 rounded-lg text-white" style={{ background: "#0B6E3E" }}><SettingsIcon size={15} /> Open Admin Panel</button>}
                {role === "chairman" && <button onClick={() => setChairmanPanelOpen(true)} className="mt-1 w-full flex items-center justify-center gap-2 text-sm font-medium px-4 py-2.5 rounded-lg text-white" style={{ background: "#C79A2E" }}><Crown size={15} /> Open Chairman Panel</button>}

                <div className="mt-4 pt-3 border-t space-y-2" style={{ borderColor: "#DCD7C7" }}>
                  <button onClick={() => setView("messages")} className="w-full flex items-center gap-2 text-sm font-medium py-2"><MessagesSquare size={16} style={{ color: "#0B6E3E" }} /> Messages</button>
                  <button onClick={() => setInfoModal("settings")} className="w-full flex items-center gap-2 text-sm font-medium py-2"><SettingsIcon size={16} style={{ color: "#0B6E3E" }} /> Settings</button>
                  <button onClick={() => setInfoModal("support")} className="w-full flex items-center gap-2 text-sm font-medium py-2"><LifeBuoy size={16} style={{ color: "#0B6E3E" }} /> Support Center</button>
                  <button onClick={() => setInfoModal("privacy")} className="w-full flex items-center gap-2 text-sm font-medium py-2"><ScrollText size={16} style={{ color: "#0B6E3E" }} /> Privacy Policy</button>
                  <button onClick={() => setInfoModal("terms")} className="w-full flex items-center gap-2 text-sm font-medium py-2"><FileText size={16} style={{ color: "#0B6E3E" }} /> Terms & Conditions</button>
                  <button onClick={logout} className="w-full flex items-center gap-2 text-sm font-medium py-2" style={{ color: "#C0392B" }}><LogOut size={16} /> Log Out</button>
                </div>
              </div>
            </div>
          </div>
        )}
      </main>

      <nav className="fixed bottom-0 left-0 right-0 z-30 flex border-t bg-white max-w-2xl mx-auto w-full" style={{ borderColor: "#DCD7C7" }}>
        {navItems.map((t) => <NavButton key={t.id} active={tab === t.id} icon={t.icon} label={t.label} onClick={() => { setTab(t.id); setViewedUser(null); }} />)}
      </nav>

      {infoModal === "privacy" && (<InfoModal title="Privacy Policy" icon={<ScrollText size={17} style={{ color: "#0B6E3E" }} />} onClose={() => setInfoModal(null)}><p className="text-sm text-neutral-600 leading-relaxed">{PRIVACY_TEXT}</p></InfoModal>)}
      {infoModal === "terms" && (<InfoModal title="Terms & Conditions" icon={<FileText size={17} style={{ color: "#0B6E3E" }} />} onClose={() => setInfoModal(null)}><p className="text-sm text-neutral-600 leading-relaxed">{TERMS_TEXT}</p></InfoModal>)}
      {infoModal === "support" && (
        <InfoModal title="Support Center" icon={<LifeBuoy size={17} style={{ color: "#0B6E3E" }} />} onClose={() => setInfoModal(null)}>
          <p className="text-xs text-neutral-500 mb-3">Send a message to the Admin. They'll see who sent it and reply here.</p>
          <textarea value={supportDraft} onChange={(e) => setSupportDraft(e.target.value)} rows={3} placeholder="How can we help?" className="w-full border rounded-lg px-3 py-2 text-sm outline-none mb-2" style={{ borderColor: "#DCD7C7" }} />
          <button onClick={submitTicket} className="text-xs font-medium px-4 py-2 rounded-lg text-white mb-4" style={{ background: "#0B6E3E" }}>Send to Admin</button>
          <div className="space-y-2">
            {supportTickets.filter((t) => t.name === myName).map((t) => (
              <div key={t.id} className="rounded-lg border p-2.5 text-xs" style={{ borderColor: "#DCD7C7" }}>
                <p className="text-neutral-700 mb-1">{t.message}</p>
                {t.reply ? <p style={{ color: "#0B6E3E" }}><strong>Admin:</strong> {t.reply}</p> : <p className="text-neutral-400">Waiting for Admin's reply...</p>}
              </div>
            ))}
          </div>
        </InfoModal>
      )}
      {infoModal === "settings" && (
        <InfoModal title="Settings" icon={<SettingsIcon size={17} style={{ color: "#0B6E3E" }} />} onClose={() => setInfoModal(null)}>
          <div className="space-y-5">
            <div>
              <SectionTitle icon={<KeyRound size={15} style={{ color: "#0B6E3E" }} />}>Change Password</SectionTitle>
              <div className="space-y-2">
                <input type="password" value={passwords.current} onChange={(e) => setPasswords((p) => ({ ...p, current: e.target.value }))} placeholder="Current password" className="w-full border rounded-lg px-3 py-2 text-sm outline-none" style={{ borderColor: "#DCD7C7" }} />
                <input type="password" value={passwords.next} onChange={(e) => setPasswords((p) => ({ ...p, next: e.target.value }))} placeholder="New password" className="w-full border rounded-lg px-3 py-2 text-sm outline-none" style={{ borderColor: "#DCD7C7" }} />
                <button className="text-xs font-medium px-4 py-2 rounded-lg text-white" style={{ background: "#0B6E3E" }}>Save Password</button>
              </div>
            </div>

            <div className="pt-4 border-t" style={{ borderColor: "#DCD7C7" }}>
              <SectionTitle icon={<MessagesSquare size={15} style={{ color: "#0B6E3E" }} />}>Who can message you</SectionTitle>
              <div className="space-y-1.5">
                {[["everyone", "Everyone"], ["friends", "Friends only (mutual follows)"], ["locked", "Locked — nobody can message me"]].map(([val, label]) => (
                  <label key={val} className="flex items-center gap-2 text-sm"><input type="radio" checked={messagePrivacy === val} onChange={() => setMessagePrivacy(val)} /> {label}</label>
                ))}
              </div>
            </div>

            <div className="pt-4 border-t" style={{ borderColor: "#DCD7C7" }}>
              <SectionTitle icon={<BadgeCheck size={15} style={{ color: "#2E7CD6" }} />}>Get Verified</SectionTitle>
              {verified ? (
                <p className="text-sm flex items-center gap-1" style={{ color: "#0B6E3E" }}><BadgeCheck size={15} /> Your account is verified.</p>
              ) : verificationRequests.find((r) => r.email === profile.email && r.status === "pending") ? (
                <p className="text-xs text-neutral-500">Your receipt was submitted — waiting for Admin to confirm.</p>
              ) : (
                <div className="space-y-2">
                  <p className="text-xs text-neutral-500">₦500 / month. Make a payment to the account below, then upload your receipt.</p>
                  <div className="rounded-lg p-3 text-sm" style={{ background: "#F7F5EE" }}>
                    <p><strong>{BANK.name}</strong></p><p>{BANK.bank}</p>
                    <p className="flex items-center gap-2">{BANK.account} <button onClick={copyBank}><Copy size={13} /></button></p>
                  </div>
                  <input value={senderName} onChange={(e) => setSenderName(e.target.value)} placeholder="Name the payment was sent under" className="w-full border rounded-lg px-3 py-2 text-sm outline-none" style={{ borderColor: "#DCD7C7" }} />
                  <label className="flex items-center gap-2 text-xs border rounded-lg px-3 py-2 cursor-pointer" style={{ borderColor: "#DCD7C7" }}>
                    <Upload size={14} /> {receiptFile ? "Receipt selected ✓" : "Upload payment receipt"}
                    <input type="file" accept="image/*" hidden onChange={onReceiptFile} />
                  </label>
                  <button disabled={!receiptFile || !senderName.trim()} onClick={submitVerification} className="w-full text-xs font-medium px-4 py-2 rounded-lg text-white disabled:opacity-50" style={{ background: "#2E7CD6" }}>I Have Made Payment</button>
                </div>
              )}
            </div>
          </div>
        </InfoModal>
      )}

      {adminOpen && role === "admin" && (
        <div className="fixed inset-0 z-40 bg-black/40 flex items-end sm:items-center justify-center">
          <div className="bg-white w-full sm:max-w-lg sm:rounded-xl rounded-t-xl max-h-[85vh] overflow-y-auto p-4 space-y-5">
            <div className="flex items-center justify-between"><p className="brz-display text-lg font-semibold">Admin Panel</p><button onClick={() => setAdminOpen(false)}><X size={18} /></button></div>

            <div>
              <SectionTitle icon={<ImageIcon size={15} style={{ color: "#0B6E3E" }} />}>Home Banners</SectionTitle>
              {banners.map((b) => (
                <div key={b.id} className="flex items-center gap-2 text-xs py-1.5 border-t" style={{ borderColor: "#DCD7C7" }}>
                  <span className="flex-1 truncate">{b.text || "(image only)"}</span>
                  <button onClick={() => { setBannerDraft(b.text || ""); setBanners((bs) => bs.filter((x) => x.id !== b.id)); }} className="px-2 py-1 rounded" style={{ background: "#F0EEE4" }}><Trash2 size={12} /></button>
                </div>
              ))}
              <div className="mt-2 space-y-1.5">
                <input value={bannerDraft} onChange={(e) => setBannerDraft(e.target.value)} className="w-full border rounded-lg px-3 py-2 text-sm outline-none" style={{ borderColor: "#DCD7C7" }} placeholder="Banner text..." />
                <label className="flex items-center gap-1.5 text-xs border rounded-lg px-3 py-2 cursor-pointer w-fit" style={{ borderColor: "#DCD7C7" }}>
                  <Camera size={13} /> {bannerImgDraft ? "Image selected ✓" : "Add banner picture"}
                  <input type="file" accept="image/*" hidden onChange={(e) => { const f = e.target.files?.[0]; if (!f) return; const r = new FileReader(); r.onload = () => setBannerImgDraft(r.result); r.readAsDataURL(f); }} />
                </label>
                <button onClick={() => { if (!bannerDraft.trim() && !bannerImgDraft) return; setBanners((bs) => [...bs, { id: Date.now(), text: bannerDraft, image: bannerImgDraft }]); setBannerDraft(""); setBannerImgDraft(null); }} className="text-xs font-medium px-3 py-1.5 rounded-lg text-white" style={{ background: "#0B6E3E" }}>Add Banner</button>
              </div>
            </div>

            <div>
              <SectionTitle icon={<Search size={15} style={{ color: "#0B6E3E" }} />}>Search Users</SectionTitle>
              <input value={adminSearch} onChange={(e) => setAdminSearch(e.target.value)} placeholder="Search by name or email..." className="w-full border rounded-lg px-3 py-2 text-sm outline-none mb-2" style={{ borderColor: "#DCD7C7" }} />
              {adminSearch && adminFilteredUsers.map((u) => (
                <div key={u.id} className="flex items-center justify-between text-sm py-2 border-t" style={{ borderColor: "#DCD7C7" }}>
                  <div><p className="font-medium flex items-center gap-1">{u.full_name || u.email}{u.verified && <BadgeCheck size={12} style={{ color: "#2E7CD6" }} />}</p><p className="text-xs text-neutral-500">{u.email}</p></div>
                  <button onClick={() => setNoticeTarget(u.full_name || u.email)} className="text-xs px-2 py-1 rounded" style={{ background: "#063C22", color: "white" }}>Target for Notice</button>
                </div>
              ))}
            </div>

            <div>
              <SectionTitle icon={<Flag size={15} style={{ color: "#C0392B" }} />}>Violation Queue</SectionTitle>
              {violationQueue.length === 0 && <p className="text-xs text-neutral-400">No reported posts.</p>}
              {violationQueue.map((f) => (
                <div key={f.id} className="flex items-center justify-between text-sm py-2 border-t" style={{ borderColor: "#DCD7C7" }}>
                  <div><p className="font-medium">{f.name}</p><p className="text-xs text-neutral-500">{f.text}</p></div>
                  <div className="flex gap-2">
                    <button onClick={() => decideViolation(f.id)} className="flex items-center gap-1 text-xs px-2 py-1 rounded" style={{ background: "#0B6E3E", color: "white" }}><CheckCircle2 size={12} /> Keep</button>
                    <button onClick={() => decideViolation(f.id)} className="flex items-center gap-1 text-xs px-2 py-1 rounded" style={{ background: "#C0392B", color: "white" }}><Ban size={12} /> Remove</button>
                  </div>
                </div>
              ))}
            </div>

            <div>
              <SectionTitle icon={<FileText size={15} style={{ color: "#0B6E3E" }} />}>Form Access Requests</SectionTitle>
              {formAccessRequests.length === 0 && <p className="text-xs text-neutral-400">No requests yet.</p>}
              {formAccessRequests.map((r) => (
                <div key={r.id} className="flex items-center justify-between text-sm py-2 border-t" style={{ borderColor: "#DCD7C7" }}>
                  <div><p className="font-medium">{r.name}</p><p className="text-xs text-neutral-500">{r.message}</p></div>
                  {r.granted ? <span className="text-xs" style={{ color: "#0B6E3E" }}>Granted ✓</span> : <button onClick={() => grantAccess(r.id)} className="text-xs px-2 py-1 rounded text-white" style={{ background: "#0B6E3E" }}>Grant Form</button>}
                </div>
              ))}
            </div>

            <div>
              <SectionTitle icon={<LifeBuoy size={15} style={{ color: "#0B6E3E" }} />}>Support Center</SectionTitle>
              {supportTickets.length === 0 && <p className="text-xs text-neutral-400">No support messages yet.</p>}
              {supportTickets.map((t) => (
                <div key={t.id} className="py-2 border-t text-sm" style={{ borderColor: "#DCD7C7" }}>
                  <p className="font-medium">{t.name}</p><p className="text-xs text-neutral-600 mb-1">{t.message}</p>
                  {t.reply ? <p className="text-xs" style={{ color: "#0B6E3E" }}>Replied: {t.reply}</p> : (
                    <div className="flex gap-2 mt-1">
                      <input value={replyDrafts[t.id] || ""} onChange={(e) => setReplyDrafts((r) => ({ ...r, [t.id]: e.target.value }))} placeholder="Type a reply..." className="flex-1 border rounded-lg px-2 py-1 text-xs outline-none" style={{ borderColor: "#DCD7C7" }} />
                      <button onClick={() => sendReply(t.id)} className="text-xs px-2 py-1 rounded text-white" style={{ background: "#0B6E3E" }}>Reply</button>
                    </div>
                  )}
                </div>
              ))}
            </div>

            <div>
              <SectionTitle icon={<BadgeCheck size={15} style={{ color: "#2E7CD6" }} />}>Verification Requests</SectionTitle>
              {verificationRequests.length === 0 && <p className="text-xs text-neutral-400">No pending requests.</p>}
              {verificationRequests.map((r) => (
                <div key={r.id} className="py-2 border-t text-sm" style={{ borderColor: "#DCD7C7" }}>
                  <div className="flex items-center justify-between">
                    <div><p className="font-medium">{r.name}</p><p className="text-xs text-neutral-500">Sender: {r.senderName}</p></div>
                    {r.status === "approved" ? <span className="text-xs" style={{ color: "#0B6E3E" }}>Verified ✓</span> : (
                      <div className="flex gap-2">
                        <button onClick={() => approveVerification(r)} className="text-xs px-2 py-1 rounded text-white" style={{ background: "#2E7CD6" }}>Approve</button>
                        <button onClick={() => setVerificationRequests((v) => v.map((x) => x.id === r.id ? { ...x, status: "rejected" } : x))} className="text-xs px-2 py-1 rounded text-white" style={{ background: "#C0392B" }}>Reject</button>
                      </div>
                    )}
                  </div>
                  {r.receipt && <img src={r.receipt} className="w-24 h-24 object-cover rounded-lg mt-1" />}
                </div>
              ))}
              <div className="mt-2 pt-2 border-t flex gap-2" style={{ borderColor: "#DCD7C7" }}>
                <input value={freeVerifyEmail} onChange={(e) => setFreeVerifyEmail(e.target.value)} placeholder="Email to verify for free..." className="flex-1 border rounded-lg px-2 py-1.5 text-xs outline-none" style={{ borderColor: "#DCD7C7" }} />
                <button onClick={giveFreeVerification} className="text-xs px-2 py-1.5 rounded text-white" style={{ background: "#0B6E3E" }}>Give Free Verification</button>
              </div>
            </div>

            <div>
              <SectionTitle icon={<Users size={15} style={{ color: "#0B6E3E" }} />}>Update Follower Counts</SectionTitle>
              {users.map((u) => (
                <div key={u.id} className="flex items-center gap-2 text-sm py-1.5 border-t" style={{ borderColor: "#DCD7C7" }}>
                  <p className="flex-1 font-medium truncate">{u.full_name || u.email}</p>
                  <input defaultValue={u.followers || 0} onChange={(e) => setFollowerEdits((f) => ({ ...f, [u.id]: e.target.value }))} className="w-20 border rounded px-2 py-1 text-xs outline-none" style={{ borderColor: "#DCD7C7" }} />
                  <button onClick={() => saveFollowerCount(u)} className="text-xs px-2 py-1 rounded text-white" style={{ background: "#0B6E3E" }}>Save</button>
                </div>
              ))}
            </div>

            <div>
              <SectionTitle icon={<Bell size={15} style={{ color: "#C79A2E" }} />}>Send Notice</SectionTitle>
              <select value={noticeTarget} onChange={(e) => setNoticeTarget(e.target.value)} className="w-full border rounded-lg px-2 py-1.5 text-xs outline-none mb-2" style={{ borderColor: "#DCD7C7" }}>
                <option value="everyone">Everyone</option>
                {users.map((u) => <option key={u.id} value={u.full_name || u.email}>{u.full_name || u.email} (private warning)</option>)}
              </select>
              <div className="flex gap-2">
                <input value={noticeText} onChange={(e) => setNoticeText(e.target.value)} placeholder="Write a notice..." className="flex-1 border rounded-lg px-3 py-2 text-sm outline-none" style={{ borderColor: "#DCD7C7" }} />
                <button onClick={sendNotice} className="text-sm font-medium px-4 py-2 rounded-lg text-white" style={{ background: "#C79A2E" }}>Send</button>
              </div>
            </div>

            <div className="pt-4 border-t" style={{ borderColor: "#DCD7C7" }}>
              <SectionTitle icon={<Crown size={15} style={{ color: "#C79A2E" }} />}>Chat with Chairman</SectionTitle>
              <div className="max-h-32 overflow-y-auto space-y-1.5 mb-2">{chatLog.map((c) => (<p key={c.id} className="text-xs"><strong style={{ color: c.from === "chairman" ? "#C79A2E" : "#0B6E3E" }}>{c.from === "chairman" ? "Chairman" : "Admin"}:</strong> {c.text}</p>))}</div>
              <div className="flex gap-2">
                <input value={adminChatDraft} onChange={(e) => setAdminChatDraft(e.target.value)} placeholder="Message the Chairman..." className="flex-1 border rounded-lg px-3 py-2 text-sm outline-none" style={{ borderColor: "#DCD7C7" }} />
                <button onClick={() => sendChat("admin")} className="text-xs px-3 py-2 rounded-lg text-white" style={{ background: "#0B6E3E" }}>Send</button>
              </div>
            </div>
          </div>
        </div>
      )}

      {chairmanPanelOpen && role === "chairman" && (
        <div className="fixed inset-0 z-40 bg-black/40 flex items-end sm:items-center justify-center">
          <div className="bg-white w-full sm:max-w-lg sm:rounded-xl rounded-t-xl max-h-[85vh] overflow-y-auto p-4 space-y-5">
            <div className="flex items-center justify-between"><p className="brz-display text-lg font-semibold flex items-center gap-1.5"><Crown size={17} style={{ color: "#C79A2E" }} /> Chairman Panel</p><button onClick={() => setChairmanPanelOpen(false)}><X size={18} /></button></div>

            <div>
              <SectionTitle icon={<Search size={15} style={{ color: "#0B6E3E" }} />}>Search Members</SectionTitle>
              <input value={chairmanSearch} onChange={(e) => setChairmanSearch(e.target.value)} placeholder="Search by name or email..." className="w-full border rounded-lg px-3 py-2 text-sm outline-none mb-2" style={{ borderColor: "#DCD7C7" }} />
              {chairmanSearch && chairmanFilteredUsers.map((u) => (
                <div key={u.id} className="flex items-center justify-between text-sm py-2 border-t" style={{ borderColor: "#DCD7C7" }}>
                  <div><p className="font-medium">{u.full_name || u.email}</p><p className="text-xs text-neutral-500">{u.followers || 0} followers</p></div>
                  <button onClick={() => setGiftTarget(u)} className="flex items-center gap-1 text-xs px-2 py-1 rounded text-white" style={{ background: "#C79A2E" }}><Gift size={12} /> Message</button>
                </div>
              ))}
              {giftTarget && (
                <div className="mt-2 rounded-lg border p-2.5" style={{ borderColor: "#C79A2E", background: "#FFFDF6" }}>
                  <p className="text-xs font-medium mb-1">Private message to {giftTarget.full_name || giftTarget.email} (members can only message you back through Admin)</p>
                  <textarea value={giftText} onChange={(e) => setGiftText(e.target.value)} rows={2} className="w-full text-xs border rounded-lg px-2 py-1.5 outline-none" style={{ borderColor: "#DCD7C7" }} />
                  <div className="flex gap-2 mt-1"><button onClick={sendGift} className="text-xs px-3 py-1.5 rounded text-white" style={{ background: "#C79A2E" }}>Send</button><button onClick={() => setGiftTarget(null)} className="text-xs px-3 py-1.5 rounded" style={{ background: "#F0EEE4" }}>Cancel</button></div>
                </div>
              )}
            </div>

            <div>
              <SectionTitle icon={<FileText size={15} style={{ color: "#0B6E3E" }} />}>Form Submissions</SectionTitle>
              {formSubmissions.length === 0 && <p className="text-xs text-neutral-400">No submissions yet.</p>}
              {formSubmissions.map((s) => (
                <div key={s.id} className="flex items-center justify-between text-sm py-2 border-t" style={{ borderColor: "#DCD7C7" }}>
                  <div><p className="font-medium">{s.title}</p><p className="text-xs text-neutral-500">{s.details}</p><p className="text-[11px] text-neutral-400">by {s.name}</p></div>
                  {s.status === "pending" ? (
                    <div className="flex gap-2">
                      <button onClick={() => decideSubmission(s.id, "approved")} className="flex items-center gap-1 text-xs px-2 py-1 rounded text-white" style={{ background: "#0B6E3E" }}><CheckCircle2 size={12} /> Approve</button>
                      <button onClick={() => decideSubmission(s.id, "rejected")} className="flex items-center gap-1 text-xs px-2 py-1 rounded text-white" style={{ background: "#C0392B" }}><XCircle size={12} /> Reject</button>
                    </div>
                  ) : <span className="text-xs" style={{ color: s.status === "approved" ? "#0B6E3E" : "#C0392B" }}>{s.status}</span>}
                </div>
              ))}
            </div>

            <div>
              <SectionTitle icon={<Bell size={15} style={{ color: "#C79A2E" }} />}>Notice to Everyone</SectionTitle>
              <div className="flex gap-2">
                <input value={chairmanNoticeText} onChange={(e) => setChairmanNoticeText(e.target.value)} placeholder="Write a notice for everyone..." className="flex-1 border rounded-lg px-3 py-2 text-sm outline-none" style={{ borderColor: "#DCD7C7" }} />
                <button onClick={sendChairmanNotice} className="text-sm font-medium px-4 py-2 rounded-lg text-white" style={{ background: "#C79A2E" }}>Send</button>
              </div>
            </div>

            <div className="pt-4 border-t" style={{ borderColor: "#DCD7C7" }}>
              <SectionTitle icon={<SettingsIcon size={15} style={{ color: "#0B6E3E" }} />}>Chat with Admin</SectionTitle>
              <div className="max-h-32 overflow-y-auto space-y-1.5 mb-2">{chatLog.map((c) => (<p key={c.id} className="text-xs"><strong style={{ color: c.from === "chairman" ? "#C79A2E" : "#0B6E3E" }}>{c.from === "chairman" ? "Chairman" : "Admin"}:</strong> {c.text}</p>))}</div>
              <div className="flex gap-2">
                <input value={chairmanChatDraft} onChange={(e) => setChairmanChatDraft(e.target.value)} placeholder="Message the Admin..." className="flex-1 border rounded-lg px-3 py-2 text-sm outline-none" style={{ borderColor: "#DCD7C7" }} />
                <button onClick={() => sendChat("chairman")} className="text-xs px-3 py-2 rounded-lg text-white" style={{ background: "#C79A2E" }}>Send</button>
              </div>
            </div>
          </div>
        </div>
      )}
      </>)}
    </div>
  );
}
