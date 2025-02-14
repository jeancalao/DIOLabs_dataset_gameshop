# Descrição
Descrito neste arquivo os scripts (fontes Python) que foram utilizados, dentro da ferramenta Claude.AI para extrair os dados solicitados nos PROMPTS.

# Prompt 1 - Verificar os top 10 produtos em vendas
´´´  
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
´´´
'''

