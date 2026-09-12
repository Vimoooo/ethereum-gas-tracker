import requests
from decimal import Decimal

RPC_URL = "https://eth.llamarpc.com"

def rpc_call(method, params=None):
    payload = {
        "jsonrpc": "2.0",
        "method": method,
        "params": params or [],
        "id": 1
    }

    response = requests.post(
        RPC_URL,
        json=payload,
        timeout=15
    )
    response.raise_for_status()

    data = response.json()

    if "error" in data:
        raise RuntimeError(data["error"])

    return data["result"]


def get_gas_price():
    result = rpc_call("eth_gasPrice")
    wei = int(result, 16)

    return Decimal(wei) / Decimal(10**9)


def estimate_cost(gas_limit, gas_price_gwei):
    gas_price_wei = gas_price_gwei * Decimal(10**9)
    total_wei = gas_price_wei * Decimal(gas_limit)

    return total_wei / Decimal(10**18)


def main():
    gas_price = get_gas_price()

    print("Ethereum Gas Tracker")
    print("-" * 30)
    print(f"Current Gas: {gas_price:.2f} Gwei\n")

    transactions = {
        "ETH Transfer": 21000,
        "ERC-20 Transfer": 65000,
        "Swap": 180000,
        "NFT Mint": 150000
    }

    print("Estimated transaction costs:\n")

    for name, gas_limit in transactions.items():
        cost = estimate_cost(
            gas_limit,
            gas_price
        )

        (
            f"{name:<18} "
            f"{cost:.6f} ETH"
        )


if __name__ == "__main__":
    main()
