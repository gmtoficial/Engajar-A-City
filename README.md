// Painel estilo dashboard completo conforme imagem de referência e requisitos adicionais

import React, { useState, useEffect } from "react";
import { Card, CardContent } from "@/components/ui/card";
import { Button } from "@/components/ui/button";
import { Input } from "@/components/ui/input";
import {
  LineChart,
  Line,
  XAxis,
  YAxis,
  Tooltip,
  ResponsiveContainer,
  CartesianGrid,
  BarChart,
  Bar,
  PieChart,
  Pie,
  Cell,
  Legend,
  Brush,
} from "recharts";

const PASTAS = ["Retenção", "Conexão", "Introdução", "Evolução", "Interação", "Expansão"];
const PREMIOS = ["VIP OURO", "VIP PRATA", "VIP PLATINA", "VIP LANÇAMENTO", "BATTLEPASS"];
const CORES = ["#9333ea", "#7f1d1d", "#1e40af", "#047857", "#78350f", "#2563eb"];
const STATUS_CORES = {
  "Horrível": "#dc2626",
  "Ruim": "#f97316",
  "Médio": "#eab308",
  "Bom": "#10b981",
  "Ótimo": "#3b82f6",
  "Excelente": "#8b5cf6"
};

const getHoje = () => new Date().toISOString().split("T")[0];
const formatarData = (str) => {
  const [y, m, d] = str.split("-");
  return `${d}/${m}/${y}`;
};
const loadData = () => JSON.parse(localStorage.getItem("painel-pastas") || "{}");
const saveData = (data) => localStorage.setItem("painel-pastas", JSON.stringify(data));

export default function Dashboard() {
  const [data, setData] = useState({});
  const [dataHoje, setDataHoje] = useState(getHoje());
  const [inputs, setInputs] = useState({});
  const [premioInput, setPremioInput] = useState({ nome: '', pasta: '', tipo: '' });
  const [premiacoes, setPremiacoes] = useState([]);
  const [mostrarFormulario, setMostrarFormulario] = useState(false);
  const [mostrarConsulta, setMostrarConsulta] = useState(false);
  const [dataConsulta, setDataConsulta] = useState(getHoje());

  useEffect(() => {
    const d = loadData();
    setData(d);
    setInputs(d[dataHoje]?.pastas || {});
    setPremiacoes(d[dataHoje]?.premiacoes || []);
  }, [dataHoje]);

  const salvar = () => {
    const novo = {
      ...data,
      [dataHoje]: {
        pastas: inputs,
        premiacoes,
      },
    };
    setData(novo);
    saveData(novo);
    setMostrarFormulario(false);
  };

  const dias = Object.keys(data).sort();
  const dadosContratacao = dias.map((dia) => {
    const d = data[dia]?.pastas || {};
    const obj = { data: formatarData(dia) };
    PASTAS.forEach((p) => {
      obj[p] = Number(d[`${p}-C`] || 0);
    });
    return obj;
  });
  const dadosRetencao = dias.map((dia) => {
    const d = data[dia]?.pastas || {};
    const obj = { data: formatarData(dia) };
    PASTAS.forEach((p) => {
      obj[p] = Number(d[`${p}-R`] || 0);
    });
    return obj;
  });

  const statusNivel = (n) => {
    if (n >= 90) return "Excelente";
    if (n >= 70) return "Ótimo";
    if (n >= 50) return "Bom";
    if (n >= 30) return "Médio";
    if (n >= 10) return "Ruim";
    return "Horrível";
  };

  const premiacoesConsulta = Object.values(data).flatMap((d) => d.premiacoes || []);
  const dadosHoje = data[dataHoje]?.pastas || {};

  return (
    <div className="bg-black text-white min-h-screen p-4 max-w-[75%] mx-auto font-sans text-sm">
      <h1 className="text-center text-2xl font-bold mb-6">CONTROLE WALLSTREET</h1>

      <div className="flex items-center gap-2 mb-4">
        <Button onClick={() => setMostrarFormulario(!mostrarFormulario)} className="bg-purple-600 px-4 py-1 rounded-xl text-sm">Preencher Dados</Button>
        <Input type="date" value={dataHoje} onChange={(e) => setDataHoje(e.target.value)} className="bg-zinc-900 text-white rounded-xl text-sm px-2 py-1 w-40" />
      </div>

      {mostrarFormulario && (
        <Card className="bg-zinc-900 mb-6">
          <CardContent className="p-4 space-y-3">
            {PASTAS.map((pasta) => (
              <div key={pasta} className="grid grid-cols-3 gap-2 items-center">
                <span className="text-white text-sm font-medium col-span-1">{pasta}</span>
                <Input type="number" placeholder="Contratados" value={inputs[`${pasta}-C`] || ''} onChange={(e) => setInputs({ ...inputs, [`${pasta}-C`]: e.target.value })} className="bg-zinc-800 text-sm px-2 py-1 rounded-md" />
                <Input type="number" placeholder="Retenção" value={inputs[`${pasta}-R`] || ''} onChange={(e) => setInputs({ ...inputs, [`${pasta}-R`]: e.target.value })} className="bg-zinc-800 text-sm px-2 py-1 rounded-md" />
              </div>
            ))}
            {[0, 1, 2].map((_, i) => (
              <div key={i} className="grid grid-cols-4 gap-2">
                <Input placeholder="Nome" value={premiacoes[i]?.nome || ''} onChange={(e) => {
                  const novos = [...premiacoes];
                  novos[i] = { ...novos[i], nome: e.target.value };
                  setPremiacoes(novos);
                }} className="bg-zinc-800 text-sm px-2 py-1 rounded-md" />
                <select className="bg-zinc-800 text-white text-sm rounded-md" value={premiacoes[i]?.pasta || ''} onChange={(e) => {
                  const novos = [...premiacoes];
                  novos[i] = { ...novos[i], pasta: e.target.value };
                  setPremiacoes(novos);
                }}>
                  <option value="">Pasta</option>
                  {PASTAS.map(p => <option key={p}>{p}</option>)}
                </select>
                <select className="bg-zinc-800 text-white text-sm rounded-md" value={premiacoes[i]?.tipo || ''} onChange={(e) => {
                  const novos = [...premiacoes];
                  novos[i] = { ...novos[i], tipo: e.target.value };
                  setPremiacoes(novos);
                }}>
                  <option value="">Prêmio</option>
                  {PREMIOS.map(p => <option key={p}>{p}</option>)}
                </select>
                <div></div>
              </div>
            ))}
            <Button onClick={salvar} className="bg-green-600 mt-2 px-4 py-1 text-sm">Salvar</Button>
          </CardContent>
        </Card>
      )}

      <Card className="bg-zinc-900 mb-4">
        <CardContent className="p-4">
          <h2 className="text-purple-400 text-sm font-semibold mb-2">Linha do tempo de Contratações</h2>
          <ResponsiveContainer width="100%" height={220}>
            <LineChart data={dadosContratacao}>
              <CartesianGrid stroke="#444" strokeWidth={1} />
              <XAxis dataKey="data" stroke="#ddd" />
              <YAxis stroke="#ddd" />
              <Tooltip />
              <Legend />
              <Brush dataKey="data" height={20} stroke="#9333ea" />
              {PASTAS.map((p, i) => <Line key={p} type="monotone" dataKey={p} stroke={CORES[i]} strokeWidth={2.5} dot={{ r: 3 }} />)}
            </LineChart>
          </ResponsiveContainer>
        </CardContent>
      </Card>

      <Card className="bg-zinc-900 mb-4">
        <CardContent className="p-4">
          <h2 className="text-purple-400 text-sm font-semibold mb-2">Linha do tempo de Retenção</h2>
          <ResponsiveContainer width="100%" height={220}>
            <LineChart data={dadosRetencao}>
              <CartesianGrid stroke="#444" strokeWidth={1} />
              <XAxis dataKey="data" stroke="#ddd" />
              <YAxis stroke="#ddd" />
              <Tooltip />
              <Legend />
              <Brush dataKey="data" height={20} stroke="#9333ea" />
              {PASTAS.map((p, i) => <Line key={p} type="monotone" dataKey={p} stroke={CORES[i]} strokeWidth={2.5} dot={{ r: 3 }} />)}
            </LineChart>
          </ResponsiveContainer>
        </CardContent>
      </Card>

      <div className="grid grid-cols-1 md:grid-cols-2 gap-4 mb-4">
        <Card className="bg-zinc-900">
          <CardContent className="p-4">
            <h2 className="text-white text-sm mb-1">Gráfico Contratações ({formatarData(dataHoje)})</h2>
            <ResponsiveContainer width="100%" height={180}>
              <BarChart data={PASTAS.map((p) => ({ pasta: p, valor: Number(dadosHoje[`${p}-C`] || 0) }))}>
                <XAxis dataKey="pasta" stroke="#aaa" />
                <YAxis stroke="#aaa" />
                <Tooltip />
                <Bar dataKey="valor" fill="#10b981" />
              </BarChart>
            </ResponsiveContainer>
          </CardContent>
        </Card>
        <Card className="bg-zinc-900">
          <CardContent className="p-4">
            <h2 className="text-white text-sm mb-1">Gráfico de Retenção ({formatarData(dataHoje)})</h2>
            <ResponsiveContainer width="100%" height={180}>
              <BarChart data={PASTAS.map((p) => ({ pasta: p, valor: Number(dadosHoje[`${p}-R`] || 0) }))}>
                <XAxis dataKey="pasta" stroke="#aaa" />
                <YAxis stroke="#aaa" />
                <Tooltip />
                <Bar dataKey="valor" fill="#f97316" />
              </BarChart>
            </ResponsiveContainer>
          </CardContent>
        </Card>
      </div>

      <Card className="bg-zinc-900 mb-4">
        <CardContent className="p-4">
          <h2 className="text-green-400 text-sm font-semibold mb-2">Status das pasta - Semanal</h2>
          {PASTAS.map((p, i) => {
            const valor = Number(dadosHoje[`${p}-C`] || 0);
            const perc = Math.min(100, (valor / 1000) * 100);
            const status = statusNivel(perc);
            return (
              <div key={p} className="mb-2">
                <div className="flex justify-between text-sm">
                  <span className="text-white font-medium">{p}</span>
                  <span className="text-gray-400">{Math.round(perc)}% - {status}</span>
                </div>
                <div className="h-2 rounded bg-zinc-800">
                  <div className="h-2 rounded" style={{ width: `${perc}%`, backgroundColor: STATUS_CORES[status] }} />
                </div>
              </div>
            );
          })}
        </CardContent>
      </Card>

      <Card className="bg-zinc-900 mb-4">
        <CardContent className="p-4">
          <h2 className="text-white text-sm font-semibold mb-2">Premiações por Pasta</h2>
          <ResponsiveContainer width="100%" height={220}>
            <PieChart>
              <Pie data={premiacoesConsulta} dataKey="tipo" nameKey="pasta" cx="50%" cy="50%" outerRadius={70} label>
                {premiacoesConsulta.map((_, i) => (
                  <Cell key={i} fill={CORES[i % CORES.length]} />
                ))}
              </Pie>
              <Tooltip />
              <Legend />
            </PieChart>
          </ResponsiveContainer>
          <div className="mt-4">
            <Button onClick={() => setMostrarConsulta(!mostrarConsulta)} className="bg-white text-black px-4 py-1 text-sm">CONSULTAR</Button>
            {mostrarConsulta && (
              <div className="mt-2">
                <Input type="date" value={dataConsulta} onChange={(e) => setDataConsulta(e.target.value)} className="bg-zinc-800 text-white text-sm" />
                <ul className="text-sm text-white mt-2">
                  {data[dataConsulta]?.premiacoes?.map((p, i) => (
                    <li key={i}>{p.nome} - {p.pasta} - {p.tipo}</li>
                  )) || <li>Nenhum registro.</li>}
                </ul>
              </div>
            )}
          </div>
        </CardContent>
      </Card>

      <p className="text-center text-xs text-zinc-500 mt-6">Todos direitos reservados - BladeHT #S3LO - 2025</p>
    </div>
  );
}
