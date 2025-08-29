import React, { useMemo, useRef, useState } from "react";
import { motion } from "framer-motion";
import { QRCodeCanvas } from "qrcode.react";
import html2canvas from "html2canvas";
import jsPDF from "jspdf";
import { Download, Printer, RotateCcw } from "lucide-react";

export default function ETicketGenerator() {
  const [form, setForm] = useState({
    company: "AXE STONE FAST CRUISE",
    routeFrom: "Banjar Nyuh – Nusa Penida Port",
    routeTo: "Sanur Port – Denpasar",
    tripType: "oneway", // 'oneway' or 'return'
    passengerName: "",
    nationality: "",
    idNumber: "",
    seat: "",
    date: new Date().toISOString().slice(0, 10),
    time: "15:15",
    returnDate: new Date().toISOString().slice(0, 10),
    returnTime: "15:15",
    adult: 1,
    child: 0,
    infant: 0,
    agent: "",
    bookingCode: "",
    notes: "Please arrive 30 minutes before departure and bring your ID.",
  });

  const [generated, setGenerated] = useState(false);
  const ticketRef = useRef(null);

  const ticketNo = useMemo(() => {
    const base = `${form.date?.replaceAll("-", "")}${form.time?.replace(":", "")}`;
    const rand = Math.random().toString(36).slice(2, 6).toUpperCase();
    return `AS-${base}-${rand}`;
  }, [form.date, form.time]);

  const bookingCodeAuto = useMemo(() => {
    if (form.bookingCode?.trim()) return form.bookingCode.trim().toUpperCase();
    const rand = Math.random().toString(36).slice(2, 8).toUpperCase();
    return `AS${rand}`;
  }, [form.bookingCode]);

  const payload = useMemo(() => {
    return {
      ticketNo,
      bookingCode: bookingCodeAuto,
      company: form.company,
      route: `${form.routeFrom} → ${form.routeTo}`,
      tripType: form.tripType,
      passengerName: form.passengerName,
      nationality: form.nationality,
      idNumber: form.idNumber,
      seat: form.seat,
      date: form.date,
      time: form.time,
      returnDate: form.tripType === 'return' ? form.returnDate : undefined,
      returnTime: form.tripType === 'return' ? form.returnTime : undefined,
      adult: form.adult,
      child: form.child,
      infant: form.infant,
      agent: form.agent,
    };
  }, [form, ticketNo, bookingCodeAuto]);

  const handleChange = (e) => {
    const { name, value } = e.target;
    setForm((f) => ({ ...f, [name]: ['adult','child','infant'].includes(name) ? Number(value) : value }));
  };

  const reset = () => {
    setGenerated(false);
    setForm((f) => ({
      ...f,
      passengerName: "",
      nationality: "",
      idNumber: "",
      seat: "",
      bookingCode: "",
      adult: 1,
      child: 0,
      infant: 0,
      agent: "",
      tripType: 'oneway'
    }));
  };

  const printTicket = () => { window.print(); };

  const downloadPDF = async () => {
    if (!ticketRef.current) return;
    const canvas = await html2canvas(ticketRef.current, { scale: 2, backgroundColor: "#ffffff" });
    const imgData = canvas.toDataURL("image/png");
    const pdf = new jsPDF({ orientation: "portrait", unit: "pt", format: "a4" });
    const pageWidth = pdf.internal.pageSize.getWidth();
    const imgWidth = pageWidth - 64;
    const imgHeight = (canvas.height * imgWidth) / canvas.width;
    pdf.addImage(imgData, "PNG", 32, 32, imgWidth, imgHeight);
    pdf.save(`${payload.ticketNo}.pdf`);
  };

  return (
    <div className="min-h-screen w-full bg-gradient-to-b from-slate-50 to-white text-slate-800">
      <div className="mx-auto grid max-w-6xl gap-6 p-4 md:grid-cols-2 md:p-8">
        {/* Left: Form */}
        <motion.div initial={{ opacity: 0, y: 10 }} animate={{ opacity: 1, y: 0 }} transition={{ duration: 0.4 }} className="space-y-4 rounded-2xl border border-slate-200 bg-white p-5 shadow-sm">
          <h1 className="text-2xl font-bold tracking-tight">Web E‑Ticket Generator</h1>
          <div className="grid grid-cols-1 gap-3 md:grid-cols-2">
            <div>
              <label className="text-xs font-medium">From</label>
              <input name="routeFrom" value={form.routeFrom} onChange={handleChange} className="mt-1 w-full rounded-xl border px-3 py-2" />
            </div>
            <div>
              <label className="text-xs font-medium">To</label>
              <input name="routeTo" value={form.routeTo} onChange={handleChange} className="mt-1 w-full rounded-xl border px-3 py-2" />
            </div>
            <div className="md:col-span-2">
              <label className="text-xs font-medium">Trip Type</label>
              <select name="tripType" value={form.tripType} onChange={handleChange} className="mt-1 w-full rounded-xl border px-3 py-2">
                <option value="oneway">One Way</option>
                <option value="return">Return</option>
              </select>
            </div>
            <div>
              <label className="text-xs font-medium">Departure Date</label>
              <input type="date" name="date" value={form.date} onChange={handleChange} className="mt-1 w-full rounded-xl border px-3 py-2" />
            </div>
            <div>
              <label className="text-xs font-medium">Departure Time</label>
              <input type="time" name="time" value={form.time} onChange={handleChange} className="mt-1 w-full rounded-xl border px-3 py-2" />
            </div>
            {form.tripType === 'return' && (
              <>
                <div>
                  <label className="text-xs font-medium">Return Date</label>
                  <input type="date" name="returnDate" value={form.returnDate} onChange={handleChange} className="mt-1 w-full rounded-xl border px-3 py-2" />
                </div>
                <div>
                  <label className="text-xs font-medium">Return Time</label>
                  <input type="time" name="returnTime" value={form.returnTime} onChange={handleChange} className="mt-1 w-full rounded-xl border px-3 py-2" />
                </div>
              </>
            )}
            <div className="md:col-span-2 border-t pt-3" />
            <div className="md:col-span-2">
              <label className="text-xs font-medium">Passenger Name</label>
              <input name="passengerName" value={form.passengerName} onChange={handleChange} className="mt-1 w-full rounded-xl border px-3 py-2" />
            </div>
            <div>
              <label className="text-xs font-medium">Nationality</label>
              <input name="nationality" value={form.nationality} onChange={handleChange} className="mt-1 w-full rounded-xl border px-3 py-2" />
            </div>
            <div>
              <label className="text-xs font-medium">ID/Passport No.</label>
              <input name="idNumber" value={form.idNumber} onChange={handleChange} className="mt-1 w-full rounded-xl border px-3 py-2" />
            </div>
            <div>
              <label className="text-xs font-medium">Seat</label>
              <input name="seat" value={form.seat} onChange={handleChange} className="mt-1 w-full rounded-xl border px-3 py-2" />
            </div>
            <div className="md:col-span-2">
              <label className="text-xs font-medium">Number of Participants</label>
              <div className="grid grid-cols-3 gap-2 mt-1">
                <input type="number" min={0} name="adult" value={form.adult} onChange={handleChange} placeholder="Adult" className="rounded-xl border px-3 py-2" />
                <input type="number" min={0} name="child" value={form.child} onChange={handleChange} placeholder="Child (4-10 yo)" className="rounded-xl border px-3 py-2" />
                <input type="number" min={0} name="infant" value={form.infant} onChange={handleChange} placeholder="Infant (0-3 yo)" className="rounded-xl border px-3 py-2" />
              </div>
            </div>
            <div className="md:col-span-2">
              <label className="text-xs font-medium">Agent</label>
              <input name="agent" value={form.agent} onChange={handleChange} className="mt-1 w-full rounded-xl border px-3 py-2" />
            </div>
          </div>

          <div className="flex flex-wrap gap-3 pt-2">
            <button onClick={() => setGenerated(true)} className="rounded-2xl bg-slate-900 px-4 py-2 text-white shadow hover:opacity-90">Generate Ticket</button>
            <button onClick={printTicket} className="inline-flex items-center gap-2 rounded-2xl border px-4 py-2"><Printer size={18}/> Print</button>
            <button onClick={downloadPDF} className="inline-flex items-center gap-2 rounded-2xl border px-4 py-2"><Download size={18}/> PDF</button>
            <button onClick={reset} className="inline-flex items-center gap-2 rounded-2xl border px-4 py-2"><RotateCcw size={18}/> Reset</button>
          </div>
        </motion.div>

        {/* Right: Ticket Preview */}
        <motion.div initial={{ opacity: 0, y: 10 }} animate={{ opacity: 1, y: 0 }} transition={{ duration: 0.4, delay: 0.05 }} className="print:shadow-none print:border-0 relative rounded-2xl border border-slate-200 bg-white p-5 shadow-sm">
          <div ref={ticketRef} id="ticket" className="mx-auto max-w-[700px] rounded-2xl border bg-white p-6 print:border-0">
            <header className="mb-4 flex items-start justify-between">
              <div>
                <div className="text-xs font-semibold tracking-wider text-sky-600">E‑TICKET</div>
                <h2 className="text-xl font-black tracking-tight">{form.company}</h2>
              </div>
              <div className="text-right">
                <div className="text-[10px] text-slate-500">Ticket No.</div>
                <div className="font-mono text-sm font-semibold">{ticketNo}</div>
                <div className="text-[10px] text-slate-500">Booking Code</div>
                <div className="font-mono text-sm font-semibold">{bookingCodeAuto}</div>
              </div>
            </header>

            <section className="grid grid-cols-2 gap-3 rounded-xl bg-slate-50 p-4">
              <div>
                <div className="text-[10px] text-slate-500">From</div>
                <div className="text-sm font-semibold">{form.routeFrom}</div>
              </div>
              <div>
                <div className="text-[10px] text-slate-500">To</div>
                <div className="text-sm font-semibold">{form.routeTo}</div>
              </div>
              <div>
                <div className="text-[10px] text-slate-500">Departure Date</div>
                <div className="text-sm font-semibold">{form.date}</div>
              </div>
              <div>
                <div className="text-[10px] text-slate-500">Departure Time</div>
                <div className="text-sm font-semibold">{form.time}</div>
              </div>
              {form.tripType === 'return' && (
                <>
                  <div>
                    <div className="text-[10px] text-slate-500">Return Date</div>
                    <div className="text-sm font-semibold">{form.returnDate}</div>
                  </div>
                  <div>
                    <div className="text-[10px] text-slate-500">Return Time</div>
                    <div className="text-sm font-semibold">{form.returnTime}</div>
                  </div>
                </>
              )}
            </section>

            <section className="mt-4 grid grid-cols-2 gap-3">
              <div>
                <div className="text-[10px] text-slate-500">Passenger</div>
                <div className="text-sm font-semibold">{form.passengerName || (generated ? "—" : "(fill form)")}</div>
              </div>
              <div>
                <div className="text-[10px] text-slate-500">Seat</div>
                <div className="text-sm font-semibold">{form.seat || (generated ? "—" : "")}</div>
              </div>
              <div>
                <div className="text-[10px] text-slate-500">Nationality</div>
                <div className="text-sm font-semibold">{form.nationality || (generated ? "—" : "")}</div>
              </div>
              <div>
                <div className="text-[10px] text-slate-500">ID / Passport</div>
                <div className="text-sm font-semibold">{form.idNumber || (generated ? "—" : "")}</div>
              </div>
              <div className="md:col-span-2">
                <div className="text-[10px] text-slate-500">Number of Participants</div>
                <div className="text-sm font-semibold">Adult: {form.adult} | Child: {form.child} | Infant: {form.infant}</div>
              </div>
              <div className="md:col-span-2">
                <div className="text-[10px] text-slate-500">Agent</div>
                <div className="text-sm font-semibold">{form.agent || (generated ? "—" : "")}</div>
              </div>
            </section>

            <section className="mt-4 text-xs text-slate-600">
              <div className="font-semibold text-slate-800 mb-2">Staff Confirmed:</div>
              <div className="h-10 border-b border-slate-400 w-1/3"></div>
            </section>

            <section className="mt-4 flex items-center gap-4">
              <div className="rounded-xl border p-3">
                <QRCodeCanvas value={JSON.stringify(payload)} includeMargin size={112} />
              </div>
              <div className="text-xs text-slate-600">
                Present this QR at boarding. Staff can scan to verify booking details.
              </div>
            </section>

            <footer className="mt-4 grid grid-cols-2 gap-3 text-[10px] text-slate-500">
              <div>
                <div className="font-semibold text-slate-700">Contact</div>
                <div>support@axestonefastcruise.com</div>
                <div>+62 8xx-xxxx-xxxx</div>
              </div>
              <div className="text-right">
                <div className="font-semibold text-slate-700">Issue Date</div>
                <div>{new Date().toLocaleString()}</div>
              </div>
            </footer>
          </div>
        </motion.div>
      </div>
    </div>
  );
}
