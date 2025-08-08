# Clinical-Dashboard
Clinical Dashboard using python,plotly,html,random forest
import pandas as pd
from dash import Dash, dcc, html, Input, Output
import plotly.express as px

# Load data
df = pd.read_csv(r"C:\Users\91970\Downloads\clinical_data.csv")

# Clean columns and strings
df.columns = [c.strip().replace(' ', '_').replace('\n', '') for c in df.columns]
df['Dead_or_Alive'] = df['Dead_or_Alive'].str.strip()
df['sex'] = df['sex'].str.strip()
df['race'] = df['race'].str.strip()
df['Stage'] = df['Stage'].astype(str).str.strip()
df['Time'] = pd.to_numeric(df['Time'], errors='coerce')

app = Dash(_name_)

# Dropdown options
sex_options = [{'label': s, 'value': s} for s in sorted(df['sex'].unique())]
race_options = [{'label': r, 'value': r} for r in sorted(df['race'].unique())]
stage_options = [{'label': st, 'value': st} for st in sorted(df['Stage'].unique())]

app.layout = html.Div([
    html.H1("Clinical Data Dashboard", style={"textAlign": "center"}),

    # Filters
    html.Div([
        html.Div([
            html.Label("Filter by Sex"),
            dcc.Dropdown(id='filter-sex', options=sex_options, multi=True, placeholder="Select sex(es)")
        ], style={"width": "30%", "display": "inline-block", "padding": "0 10px"}),

        html.Div([
            html.Label("Filter by Race"),
            dcc.Dropdown(id='filter-race', options=race_options, multi=True, placeholder="Select race(s)")
        ], style={"width": "30%", "display": "inline-block", "padding": "0 10px"}),

        html.Div([
            html.Label("Filter by Stage"),
            dcc.Dropdown(id='filter-stage', options=stage_options, multi=True, placeholder="Select stage(s)")
        ], style={"width": "30%", "display": "inline-block", "padding": "0 10px"}),
    ], style={"padding": "10px 0", "borderBottom": "2px solid #ccc"}),

    # KPIs container
    html.Div(id='kpis', style={"textAlign": "center", "padding": "20px 0", "fontSize": 22, "display": "flex", "justifyContent": "space-around"}),

    # Graphs
    dcc.Graph(id='graph-sex'),
    dcc.Graph(id='graph-race'),
    dcc.Graph(id='graph-stage'),
    dcc.Graph(id='graph-time'),
    dcc.Graph(id='graph-ecdf'),
])


@app.callback(
    [
        Output('kpis', 'children'),
        Output('graph-sex', 'figure'),
        Output('graph-race', 'figure'),
        Output('graph-stage', 'figure'),
        Output('graph-time', 'figure'),
        Output('graph-ecdf', 'figure'),
    ],
    [
        Input('filter-sex', 'value'),
        Input('filter-race', 'value'),
        Input('filter-stage', 'value'),
    ]
)
def update_dashboard(selected_sex, selected_race, selected_stage):
    # Filter data based on selection or use all
    dff = df.copy()
    if selected_sex:
        dff = dff[dff['sex'].isin(selected_sex)]
    if selected_race:
        dff = dff[dff['race'].isin(selected_race)]
    if selected_stage:
        dff = dff[dff['Stage'].isin(selected_stage)]

    total = len(dff)
    dead = (dff['Dead_or_Alive'].str.lower() == 'dead').sum()
    alive = (dff['Dead_or_Alive'].str.lower() == 'alive').sum()
    mean_time = round(dff['Time'].mean(), 1) if total > 0 else 0

    # Handle empty filtered data gracefully
    if total == 0:
        no_data_fig = {
            "layout": {
                "xaxis": {"visible": False},
                "yaxis": {"visible": False},
                "annotations": [{
                    "text": "No data to display for selected filters.",
                    "xref": "paper",
                    "yref": "paper",
                    "showarrow": False,
                    "font": {"size": 20}
                }]
            }
        }
        kpis = [
            html.Div("No data to display with current filters.", style={"color": "red", "fontWeight": "bold", "fontSize": 24, "width": "100%", "textAlign": "center"})
        ]
        return kpis, no_data_fig, no_data_fig, no_data_fig, no_data_fig, no_data_fig

    # KPIs display
    kpis = [
        html.Div([
            html.Div("Total Patients", style={"fontWeight": "bold"}),
            html.Div(f"{total}")
        ], style={"width": "20%"}),
        html.Div([
            html.Div("Alive", style={"fontWeight": "bold", "color": "green"}),
            html.Div(f"{alive}")
        ], style={"width": "20%"}),
        html.Div([
            html.Div("Dead", style={"fontWeight": "bold", "color": "red"}),
            html.Div(f"{dead}")
        ], style={"width": "20%"}),
        html.Div([
            html.Div("Avg Follow-up (days)", style={"fontWeight": "bold"}),
            html.Div(f"{mean_time}")
        ], style={"width": "20%"}),
    ]

    # Build figures
    fig_sex = px.histogram(dff, x='sex', color='Dead_or_Alive',
                          barmode='group', title='Patients by Sex and Outcome',
                          color_discrete_map={'Alive':'green','Dead':'red'})
    fig_sex.update_layout(transition_duration=500)

    fig_race = px.histogram(dff, x='race', color='Dead_or_Alive',
                           barmode='group', title='Patients by Race and Outcome',
                           color_discrete_map={'Alive':'green','Dead':'red'})
    fig_race.update_layout(transition_duration=500)

    fig_stage = px.histogram(dff, x='Stage', color='Dead_or_Alive',
                            barmode='group', title='Patients by Stage and Outcome',
                            color_discrete_map={'Alive':'green','Dead':'red'})
    fig_stage.update_layout(transition_duration=500)

    fig_time = px.histogram(dff, x='Time', color='Dead_or_Alive', nbins=20,
                           barmode='overlay', title='Follow-up Time by Outcome',
                           color_discrete_map={'Alive':'green','Dead':'red'})
    fig_time.update_layout(transition_duration=500)

    fig_ecdf = px.ecdf(dff, x="Time", color="Dead_or_Alive", title="ECDF of Follow-up Time by Outcome",
                      color_discrete_map={'Alive':'green','Dead':'red'})
    fig_ecdf.update_layout(transition_duration=500)

    return kpis, fig_sex, fig_race, fig_stage, fig_time, fig_ecdf


if _name_ == '_main_':
    app.run(debug=True)


    
    clinical-dashboard/
│
├── app.py
├── requirements.txt
├── README.md
│
├── model/
│   └── data_loader.py
│
├── domain/
│   ├── filters.py
│   ├── metrics.py
│   └── figures.py
│
├── view/
│   └── layout.py
│
├── controller/
│   └── callbacks.py
│
├── utils/
│   └── config.py
│
└── assets/
    └── custom.css  # Optional styling
