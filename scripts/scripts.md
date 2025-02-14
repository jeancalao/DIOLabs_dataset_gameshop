# Descrição
Descrito neste arquivo os scripts (fontes Python) que foram utilizados, dentro da ferramenta Claude.AI para extrair os dados solicitados nos PROMPTS.

## Prompt 1 - Verificar os top 10 produtos em vendas
```  
import React, { useState, useEffect } from 'react';
import Papa from 'papaparse';

const Top10SalesTable = () => {
  const [salesData, setSalesData] = useState([]);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const loadData = async () => {
      try {
        const fileContent = await window.fs.readFile('Meganium_Sales_data.csv', { encoding: 'utf8' });
        
        const parsedData = Papa.parse(fileContent, {
          delimiter: ";",
          header: true,
          skipEmptyLines: true,
          dynamicTyping: true
        });

        const productSummary = parsedData.data.reduce((acc, row) => {
          const product = row.product_sold;
          if (!acc[product]) {
            acc[product] = {
              produto: product,
              quantidade: 0,
              valor_total: 0,
              valor_medio: 0
            };
          }
          acc[product].quantidade += Number(row.quantity) || 0;
          acc[product].valor_total += Number(row.total_price) || 0;
          return acc;
        }, {});

        // Calcular valor médio e ordenar por valor total
        Object.values(productSummary).forEach(product => {
          product.valor_medio = product.valor_total / product.quantidade;
        });

        const top10Products = Object.values(productSummary)
          .sort((a, b) => b.valor_total - a.valor_total)
          .slice(0, 10);

        setSalesData(top10Products);
        setLoading(false);
      } catch (error) {
        console.error('Erro ao carregar dados:', error);
        setLoading(false);
      }
    };

    loadData();
  }, []);

  if (loading) {
    return <div className="p-4">Carregando dados...</div>;
  }

  return (
    <div className="p-4">
      <h2 className="text-xl font-bold mb-4">Top 10 Produtos em Vendas - Megaliom</h2>
      <div className="overflow-x-auto">
        <table className="min-w-full bg-white border border-gray-300">
          <thead>
            <tr className="bg-gray-100">
              <th className="px-6 py-3 border-b text-left">Produto</th>
              <th className="px-6 py-3 border-b text-right">Quantidade</th>
              <th className="px-6 py-3 border-b text-right">Valor Total (R$)</th>
              <th className="px-6 py-3 border-b text-right">Valor Médio (R$)</th>
            </tr>
          </thead>
          <tbody>
            {salesData.map((item, index) => (
              <tr key={index} className={index % 2 === 0 ? 'bg-gray-50' : 'bg-white'}>
                <td className="px-6 py-4 border-b">{item.produto}</td>
                <td className="px-6 py-4 border-b text-right">
                  {item.quantidade.toLocaleString('pt-BR')}
                </td>
                <td className="px-6 py-4 border-b text-right">
                  {item.valor_total.toLocaleString('pt-BR', {
                    minimumFractionDigits: 2,
                    maximumFractionDigits: 2
                  })}
                </td>
                <td className="px-6 py-4 border-b text-right">
                  {(item.valor_total / item.quantidade).toLocaleString('pt-BR', {
                    minimumFractionDigits: 2,
                    maximumFractionDigits: 2
                  })}
                </td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>
    </div>
  );
};

export default Top10SalesTable;
```
## Prompt 2 - Listar os top 10 produtos, convertendo os valores para reais
Neste script, foi solicitado para converter os valores das moedas estrangeiras em Reais, considerando para dólar: R$ 5,70; Euro: R$ 6,02; Libra Esterlina: R$ 7,03.

```
import React, { useState, useEffect } from 'react';
import Papa from 'papaparse';

const Top10SalesTable = () => {
  const [salesData, setSalesData] = useState([]);
  const [loading, setLoading] = useState(true);
  const [showDetails, setShowDetails] = useState({});

  const exchangeRates = {
    'USD': 5.70,
    'EUR': 6.02,
    'GBP': 7.03,
    'BRL': 1.00,
  };

  useEffect(() => {
    const loadData = async () => {
      try {
        const fileContent = await window.fs.readFile('Meganium_Sales_data.csv', { encoding: 'utf8' });
        
        const parsedData = Papa.parse(fileContent, {
          delimiter: ";",
          header: true,
          skipEmptyLines: true,
          dynamicTyping: true
        });

        const productSummary = parsedData.data.reduce((acc, row) => {
          const product = row.product_sold;
          const currency = row.currency;
          const exchangeRate = exchangeRates[currency] || 1;
          const valueInBRL = (Number(row.total_price) || 0) * exchangeRate;

          if (!acc[product]) {
            acc[product] = {
              produto: product,
              quantidade: 0,
              valor_total_brl: 0,
              vendas_por_moeda: {}
            };
          }
          
          acc[product].quantidade += Number(row.quantity) || 0;
          acc[product].valor_total_brl += valueInBRL;
          
          if (!acc[product].vendas_por_moeda[currency]) {
            acc[product].vendas_por_moeda[currency] = {
              quantidade: 0,
              valor_original: 0,
              valor_brl: 0
            };
          }
          acc[product].vendas_por_moeda[currency].quantidade += Number(row.quantity) || 0;
          acc[product].vendas_por_moeda[currency].valor_original += Number(row.total_price) || 0;
          acc[product].vendas_por_moeda[currency].valor_brl += valueInBRL;
          
          return acc;
        }, {});

        const top10Products = Object.values(productSummary)
          .sort((a, b) => b.valor_total_brl - a.valor_total_brl)
          .slice(0, 10);

        setSalesData(top10Products);
        setLoading(false);
      } catch (error) {
        console.error('Erro ao carregar dados:', error);
        setLoading(false);
      }
    };

    loadData();
  }, []);

  const toggleDetails = (index) => {
    setShowDetails(prev => ({
      ...prev,
      [index]: !prev[index]
    }));
  };

  if (loading) {
    return <div className="p-4">Carregando dados...</div>;
  }

  return (
    <div className="p-4">
      <h2 className="text-xl font-bold mb-4">Top 10 Produtos em Vendas - Megaliom</h2>
      <div className="overflow-x-auto">
        <table className="min-w-full bg-white border border-gray-300">
          <thead>
            <tr className="bg-gray-100">
              <th className="px-6 py-3 border-b text-left">Produto</th>
              <th className="px-6 py-3 border-b text-right">Quantidade Total</th>
              <th className="px-6 py-3 border-b text-right">Valor Total (R$)</th>
              <th className="px-6 py-3 border-b text-right">Valor Médio (R$)</th>
              <th className="px-6 py-3 border-b text-center">Detalhes</th>
            </tr>
          </thead>
          <tbody>
            {salesData.map((item, index) => (
              <React.Fragment key={index}>
                <tr className={index % 2 === 0 ? 'bg-gray-50' : 'bg-white'}>
                  <td className="px-6 py-4 border-b">{item.produto}</td>
                  <td className="px-6 py-4 border-b text-right">
                    {item.quantidade.toLocaleString('pt-BR')}
                  </td>
                  <td className="px-6 py-4 border-b text-right">
                    {item.valor_total_brl.toLocaleString('pt-BR', {
                      minimumFractionDigits: 2,
                      maximumFractionDigits: 2
                    })}
                  </td>
                  <td className="px-6 py-4 border-b text-right">
                    {(item.valor_total_brl / item.quantidade).toLocaleString('pt-BR', {
                      minimumFractionDigits: 2,
                      maximumFractionDigits: 2
                    })}
                  </td>
                  <td className="px-6 py-4 border-b text-center">
                    <button 
                      onClick={() => toggleDetails(index)}
                      className="px-3 py-1 text-sm text-blue-600 hover:text-blue-800"
                    >
                      {showDetails[index] ? 'Ocultar' : 'Mostrar'}
                    </button>
                  </td>
                </tr>
                {showDetails[index] && (
                  <tr>
                    <td colSpan="5" className="px-6 py-4 border-b bg-gray-50">
                      <div className="text-sm">
                        <h4 className="font-bold mb-2">Detalhamento por Moeda:</h4>
                        <table className="w-full">
                          <thead>
                            <tr className="bg-gray-100">
                              <th className="px-4 py-2 text-left">Moeda</th>
                              <th className="px-4 py-2 text-right">Quantidade</th>
                              <th className="px-4 py-2 text-right">Valor Original</th>
                              <th className="px-4 py-2 text-right">Valor em R$</th>
                            </tr>
                          </thead>
                          <tbody>
                            {Object.entries(item.vendas_por_moeda).map(([currency, data]) => (
                              <tr key={currency}>
                                <td className="px-4 py-2">{currency}</td>
                                <td className="px-4 py-2 text-right">{data.quantidade}</td>
                                <td className="px-4 py-2 text-right">
                                  {data.valor_original.toLocaleString('pt-BR', {
                                    minimumFractionDigits: 2,
                                    maximumFractionDigits: 2
                                  })}
                                </td>
                                <td className="px-4 py-2 text-right">
                                  {data.valor_brl.toLocaleString('pt-BR', {
                                    minimumFractionDigits: 2,
                                    maximumFractionDigits: 2
                                  })}
                                </td>
                              </tr>
                            ))}
                          </tbody>
                        </table>
                      </div>
                    </td>
                  </tr>
                )}
              </React.Fragment>
            ))}
          </tbody>
        </table>
      </div>
    </div>
  );
};

export default Top10SalesTable;
```
## Prompt 3 - Listar os Países com maiores volumes de vendas, convertendo os valores para R$
```
import React, { useState, useEffect } from 'react';
import Papa from 'papaparse';

const CountrySalesTable = () => {
  const [salesData, setSalesData] = useState([]);
  const [loading, setLoading] = useState(true);
  const [showDetails, setShowDetails] = useState({});

  const exchangeRates = {
    'USD': 5.76,
    'EUR': 6.02,
    'GBP': 7.03,
    'BRL': 1.00,
  };

  useEffect(() => {
    const loadData = async () => {
      try {
        const fileContent = await window.fs.readFile('Meganium_Sales_data.csv', { encoding: 'utf8' });
        
        const parsedData = Papa.parse(fileContent, {
          delimiter: ";",
          header: true,
          skipEmptyLines: true,
          dynamicTyping: true
        });

        const countrySummary = parsedData.data.reduce((acc, row) => {
          const country = row.delivery_country;
          const currency = row.currency;
          const exchangeRate = exchangeRates[currency] || 1;
          const valueInBRL = (Number(row.total_price) || 0) * exchangeRate;

          if (!acc[country]) {
            acc[country] = {
              pais: country,
              quantidade: 0,
              valor_total_brl: 0,
              vendas_por_moeda: {}
            };
          }
          
          acc[country].quantidade += Number(row.quantity) || 0;
          acc[country].valor_total_brl += valueInBRL;
          
          if (!acc[country].vendas_por_moeda[currency]) {
            acc[country].vendas_por_moeda[currency] = {
              quantidade: 0,
              valor_original: 0,
              valor_brl: 0
            };
          }
          acc[country].vendas_por_moeda[currency].quantidade += Number(row.quantity) || 0;
          acc[country].vendas_por_moeda[currency].valor_original += Number(row.total_price) || 0;
          acc[country].vendas_por_moeda[currency].valor_brl += valueInBRL;
          
          return acc;
        }, {});

        const sortedCountries = Object.values(countrySummary)
          .sort((a, b) => b.valor_total_brl - a.valor_total_brl);

        setSalesData(sortedCountries);
        setLoading(false);
      } catch (error) {
        console.error('Erro ao carregar dados:', error);
        setLoading(false);
      }
    };

    loadData();
  }, []);

  const toggleDetails = (index) => {
    setShowDetails(prev => ({
      ...prev,
      [index]: !prev[index]
    }));
  };

  if (loading) {
    return <div className="p-4">Carregando dados...</div>;
  }

  const totalVendas = salesData.reduce((acc, item) => acc + item.valor_total_brl, 0);
  const totalQuantidade = salesData.reduce((acc, item) => acc + item.quantidade, 0);

  return (
    <div className="p-4">
      <h2 className="text-xl font-bold mb-4">Vendas por País - Megaliom</h2>
      <div className="mb-4 p-4 bg-blue-50 rounded-lg">
        <h3 className="text-lg font-semibold mb-2">Resumo Geral</h3>
        <div className="grid grid-cols-2 gap-4">
          <div>
            <p className="text-sm text-gray-600">Total de Vendas (R$)</p>
            <p className="text-lg font-bold">{totalVendas.toLocaleString('pt-BR', {
              minimumFractionDigits: 2,
              maximumFractionDigits: 2
            })}</p>
          </div>
          <div>
            <p className="text-sm text-gray-600">Total de Unidades</p>
            <p className="text-lg font-bold">{totalQuantidade.toLocaleString('pt-BR')}</p>
          </div>
        </div>
      </div>
      <div className="overflow-x-auto">
        <table className="min-w-full bg-white border border-gray-300">
          <thead>
            <tr className="bg-gray-100">
              <th className="px-6 py-3 border-b text-left">País</th>
              <th className="px-6 py-3 border-b text-right">Quantidade</th>
              <th className="px-6 py-3 border-b text-right">Valor Total (R$)</th>
              <th className="px-6 py-3 border-b text-right">Ticket Médio (R$)</th>
              <th className="px-6 py-3 border-b text-center">Detalhes</th>
            </tr>
          </thead>
          <tbody>
            {salesData.map((item, index) => (
              <React.Fragment key={index}>
                <tr className={index % 2 === 0 ? 'bg-gray-50' : 'bg-white'}>
                  <td className="px-6 py-4 border-b font-medium">{item.pais}</td>
                  <td className="px-6 py-4 border-b text-right">
                    {item.quantidade.toLocaleString('pt-BR')}
                  </td>
                  <td className="px-6 py-4 border-b text-right">
                    {item.valor_total_brl.toLocaleString('pt-BR', {
                      minimumFractionDigits: 2,
                      maximumFractionDigits: 2
                    })}
                  </td>
                  <td className="px-6 py-4 border-b text-right">
                    {(item.valor_total_brl / item.quantidade).toLocaleString('pt-BR', {
                      minimumFractionDigits: 2,
                      maximumFractionDigits: 2
                    })}
                  </td>
                  <td className="px-6 py-4 border-b text-center">
                    <button 
                      onClick={() => toggleDetails(index)}
                      className="px-3 py-1 text-sm text-blue-600 hover:text-blue-800"
                    >
                      {showDetails[index] ? 'Ocultar' : 'Mostrar'}
                    </button>
                  </td>
                </tr>
                {showDetails[index] && (
                  <tr>
                    <td colSpan="5" className="px-6 py-4 border-b bg-gray-50">
                      <div className="text-sm">
                        <h4 className="font-bold mb-2">Detalhamento por Moeda:</h4>
                        <table className="w-full">
                          <thead>
                            <tr className="bg-gray-100">
                              <th className="px-4 py-2 text-left">Moeda</th>
                              <th className="px-4 py-2 text-right">Quantidade</th>
                              <th className="px-4 py-2 text-right">Valor Original</th>
                              <th className="px-4 py-2 text-right">Valor em R$</th>
                            </tr>
                          </thead>
                          <tbody>
                            {Object.entries(item.vendas_por_moeda).map(([currency, data]) => (
                              <tr key={currency}>
                                <td className="px-4 py-2">{currency}</td>
                                <td className="px-4 py-2 text-right">{data.quantidade}</td>
                                <td className="px-4 py-2 text-right">
                                  {data.valor_original.toLocaleString('pt-BR', {
                                    minimumFractionDigits: 2,
                                    maximumFractionDigits: 2
                                  })}
                                </td>
                                <td className="px-4 py-2 text-right">
                                  {data.valor_brl.toLocaleString('pt-BR', {
                                    minimumFractionDigits: 2,
                                    maximumFractionDigits: 2
                                  })}
                                </td>
                              </tr>
                            ))}
                          </tbody>
                        </table>
                      </div>
                    </td>
                  </tr>
                )}
              </React.Fragment>
            ))}
          </tbody>
        </table>
      </div>
    </div>
  );
};

export default CountrySalesTable;
```
## Prompt 4 - Comparativo de ticket médio dos produtos
```
import React from 'react';

const TicketMedioComparison = () => {
  const produtos = [
    {
      nome: "MEGANIUM RG353M",
      ticketMedio: 656.93,
      quantidade: 29,
      ticketPorMoeda: {
        USD: 633.60,
        EUR: 662.20,
        GBP: 773.30
      }
    },
    {
      nome: "NEW MEGANIUM RG 40XXV",
      ticketMedio: 633.24,
      quantidade: 41,
      ticketPorMoeda: {
        USD: 576.00,
        EUR: 602.00,
        GBP: 703.00
      }
    },
    {
      nome: "NEW MEGANIUM RG35XX",
      ticketMedio: 554.13,
      quantidade: 36,
      ticketPorMoeda: {
        USD: 518.40,
        EUR: 541.80,
        GBP: 632.70
      }
    },
    {
      nome: "NEW MEGANIUM RG CubeXX",
      ticketMedio: 493.20,
      quantidade: 36,
      ticketPorMoeda: {
        USD: 460.80,
        EUR: 481.60,
        GBP: 562.40
      }
    },
    {
      nome: "NEW MEGANIUM RG28XX",
      ticketMedio: 433.40,
      quantidade: 36,
      ticketPorMoeda: {
        USD: 403.20,
        EUR: 421.40,
        GBP: 492.10
      }
    }
  ];

  return (
    <div className="p-4">
      <h2 className="text-xl font-bold mb-4">Comparativo de Ticket Médio por Produto</h2>
      <div className="overflow-x-auto">
        <table className="min-w-full bg-white border border-gray-300">
          <thead>
            <tr className="bg-gray-100">
              <th className="px-6 py-3 border-b text-left">Produto</th>
              <th className="px-6 py-3 border-b text-right">Ticket Médio (R$)</th>
              <th className="px-6 py-3 border-b text-right">Quantidade</th>
              <th className="px-6 py-3 border-b text-center">Ticket por Moeda (R$)</th>
            </tr>
          </thead>
          <tbody>
            {produtos.map((produto, index) => (
              <tr key={index} className={index % 2 === 0 ? 'bg-gray-50' : 'bg-white'}>
                <td className="px-6 py-4 border-b font-medium">{produto.nome}</td>
                <td className="px-6 py-4 border-b text-right font-bold">
                  {produto.ticketMedio.toLocaleString('pt-BR', {
                    minimumFractionDigits: 2,
                    maximumFractionDigits: 2
                  })}
                </td>
                <td className="px-6 py-4 border-b text-right">
                  {produto.quantidade}
                </td>
                <td className="px-6 py-4 border-b">
                  <div className="flex justify-between text-sm">
                    <span className="text-blue-600">
                      USD: {produto.ticketPorMoeda.USD.toLocaleString('pt-BR', {
                        minimumFractionDigits: 2,
                        maximumFractionDigits: 2
                      })}
                    </span>
                    <span className="text-green-600">
                      EUR: {produto.ticketPorMoeda.EUR.toLocaleString('pt-BR', {
                        minimumFractionDigits: 2,
                        maximumFractionDigits: 2
                      })}
                    </span>
                    <span className="text-purple-600">
                      GBP: {produto.ticketPorMoeda.GBP.toLocaleString('pt-BR', {
                        minimumFractionDigits: 2,
                        maximumFractionDigits: 2
                      })}
                    </span>
                  </div>
                </td>
              </tr>
            ))}
          </tbody>
        </table>
      </div>
    </div>
  );
};

export default TicketMedioComparison;
```
## Prompt 5 - Elaboração de Plano de Ação para o produto menos vendido
```
import React, { useState } from 'react';

const PlanoEstrategico = () => {
  const [filtroAtivo, setFiltroAtivo] = useState('todos');

  const acoes = {
    'Curto Prazo (3 meses)': [
      {
        area: 'Preço e Promoção',
        acao: 'Implementar programa de descontos por volume',
        impacto: 'Alto',
        prioridade: 'Alta',
        mercados: 'Todos'
      },
      {
        area: 'Capacitação',
        acao: 'Treinar equipe de vendas sobre diferenciais do produto',
        impacto: 'Médio',
        prioridade: 'Alta',
        mercados: 'Todos'
      },
      {
        area: 'Marketing',
        acao: 'Lançar campanhas promocionais nos mercados principais',
        impacto: 'Alto',
        prioridade: 'Alta',
        mercados: 'Japão, França'
      },
      {
        area: 'Vendas',
        acao: 'Criar programa de incentivo para vendedores',
        impacto: 'Médio',
        prioridade: 'Média',
        mercados: 'Todos'
      }
    ],
    'Médio Prazo (6 meses)': [
      {
        area: 'Marketing',
        acao: 'Desenvolver materiais específicos por região',
        impacto: 'Alto',
        prioridade: 'Média',
        mercados: 'Todos'
      },
      {
        area: 'Vendas',
        acao: 'Implementar programa de fidelidade',
        impacto: 'Alto',
        prioridade: 'Média',
        mercados: 'Europa, Japão'
      },
      {
        area: 'Parcerias',
        acao: 'Expandir rede de revendedores',
        impacto: 'Alto',
        prioridade: 'Média',
        mercados: 'Alemanha, Austrália'
      },
      {
        area: 'Produto',
        acao: 'Desenvolver bundles com produtos complementares',
        impacto: 'Médio',
        prioridade: 'Média',
        mercados: 'Todos'
      }
    ],
    'Longo Prazo (12 meses)': [
      {
        area: 'Expansão',
        acao: 'Avaliar entrada em novos mercados',
        impacto: 'Alto',
        prioridade: 'Baixa',
        mercados: 'Novos'
      },
      {
        area: 'Produto',
        acao: 'Desenvolver versões específicas por mercado',
        impacto: 'Alto',
        prioridade: 'Média',
        mercados: 'Todos'
      },
      {
        area: 'CRM',
        acao: 'Implementar programa de relacionamento',
        impacto: 'Alto',
        prioridade: 'Média',
        mercados: 'Todos'
      },
      {
        area: 'Operacional',
        acao: 'Otimizar gestão de estoque por região',
        impacto: 'Médio',
        prioridade: 'Alta',
        mercados: 'Todos'
      }
    ]
  };

  const prazos = ['Curto Prazo (3 meses)', 'Médio Prazo (6 meses)', 'Longo Prazo (12 meses)'];

  return (
    <div className="p-4">
      <h2 className="text-xl font-bold mb-4">Plano de Ação Estratégico - MEGANIUM RG353M</h2>
      
      <div className="mb-4">
        <label className="mr-2 font-medium">Filtrar por prazo:</label>
        <select 
          className="border p-2 rounded"
          value={filtroAtivo}
          onChange={(e) => setFiltroAtivo(e.target.value)}
        >
          <option value="todos">Todos os prazos</option>
          {prazos.map(prazo => (
            <option key={prazo} value={prazo}>{prazo}</option>
          ))}
        </select>
      </div>

      {prazos.map(prazo => (
        (filtroAtivo === 'todos' || filtroAtivo === prazo) && (
          <div key={prazo} className="mb-8">
            <h3 className="text-lg font-semibold mb-3 bg-gray-100 p-2">{prazo}</h3>
            <div className="overflow-x-auto">
              <table className="min-w-full bg-white border border-gray-300">
                <thead>
                  <tr className="bg-gray-50">
                    <th className="px-6 py-3 border-b text-left">Área</th>
                    <th className="px-6 py-3 border-b text-left">Ação</th>
                    <th className="px-6 py-3 border-b text-center">Impacto</th>
                    <th className="px-6 py-3 border-b text-center">Prioridade</th>
                    <th className="px-6 py-3 border-b text-left">Mercados</th>
                  </tr>
                </thead>
                <tbody>
                  {acoes[prazo].map((acao, index) => (
                    <tr key={index} className={index % 2 === 0 ? 'bg-white' : 'bg-gray-50'}>
                      <td className="px-6 py-4 border-b">{acao.area}</td>
                      <td className="px-6 py-4 border-b">{acao.acao}</td>
                      <td className="px-6 py-4 border-b text-center">
                        <span className={`px-2 py-1 rounded text-sm ${
                          acao.impacto === 'Alto' ? 'bg-green-100 text-green-800' :
                          acao.impacto === 'Médio' ? 'bg-yellow-100 text-yellow-800' :
                          'bg-red-100 text-red-800'
                        }`}>
                          {acao.impacto}
                        </span>
                      </td>
                      <td className="px-6 py-4 border-b text-center">
                        <span className={`px-2 py-1 rounded text-sm ${
                          acao.prioridade === 'Alta' ? 'bg-red-100 text-red-800' :
                          acao.prioridade === 'Média' ? 'bg-yellow-100 text-yellow-800' :
                          'bg-green-100 text-green-800'
                        }`}>
                          {acao.prioridade}
                        </span>
                      </td>
                      <td className="px-6 py-4 border-b">{acao.mercados}</td>
                    </tr>
                  ))}
                </tbody>
              </table>
            </div>
          </div>
        )
      ))}
    </div>
  );
};

export default PlanoEstrategico;
```

