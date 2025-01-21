# Azure SQL Serverless Cost Calculator

An interactive cost calculator for Azure SQL Database Serverless. Calculate estimated monthly costs based on vCores, active hours, and auto-pause settings.

## Access

Simply visit: [https://libsman.github.io/azure-sql-serverless-costcalculator/](https://libsman.github.io/azure-sql-serverless-costcalculator/)

## Features

- Set minimum and maximum vCores (0.25 to 80)
- Define active hours (1 to 24)
- Auto-pause option (activates after one hour of inactivity)
- Customizable cost factor per second
- Interactive cost graph
- Responsive design for desktop and mobile

## When to Use This Calculator

- When planning a new Azure SQL Database Serverless
- For estimating costs of different configurations
- To optimize existing database resources
- When comparing different scaling options

## How to Use

1. **Min vCores**: Set the minimum number of vCores (at least 1/8 of Max vCores)
2. **Max vCores**: Choose the maximum number of vCores
3. **Active Hours**: Enter the expected active hours per day
4. **Auto-Pause**: Enable/disable automatic pausing
5. **Cost Factor**: Adjust the cost factor per second if needed

The graph and cost estimation will update automatically.

## Development

```bash
# Clone repository
git clone https://github.com/libsman/azure-sql-serverless-costcalculator.git

# Navigate to cloned directory
cd azure-sql-serverless-costcalculator

# Install dependencies
npm install

# Start development server
npm run dev

# Enjoy :)