from sage.graphs.graph import Graph
from sage.graphs.digraph import DiGraph

#Exercice 1 Question 1 --------------------------------------------------------------------------------

class DFS_State:
    def __init__(self):
        self.visite = set()
        self.parent = {}
        self.temps_decouverte = {}
        self.fin_decouverte = {}
        self.temps_actuel = 0

def DFS(Graphe, sommet_depart, state):
    state.visite.add(sommet_depart)
    state.temps_decouverte[sommet_depart] = state.fin_decouverte[sommet_depart] = state.temps_actuel
    state.temps_actuel += 1

    for voisin in Graphe.neighbors(sommet_depart):
        if voisin not in state.visite:
            state.parent[voisin] = sommet_depart
            DFS(Graphe, voisin, state)
            state.fin_decouverte[sommet_depart] = min(state.fin_decouverte[sommet_depart], state.fin_decouverte[voisin])
        elif sommet_depart in state.parent and voisin != state.parent[sommet_depart]:
            state.fin_decouverte[sommet_depart] = min(state.fin_decouverte[sommet_depart], state.temps_decouverte[voisin])

def Decomposition(Graphe):
    #vérifier si le Graphe est décomposable :
    if not Est_Decomposable(Graphe):
        return False
    #vérifier si le Graphe est connexe :
    sommet_depart = next(iter(Graphe.vertex_iterator()))
    if not Est_connexe(Graphe, sommet_depart):
        print("Graphe est déconnecté")
        return False
    #Effectuer le parcours en profondeur sur le Graphe
    state = DFS_State()
    DFS(Graphe, sommet_depart, state)
    #Decomposer le graphe en composantes 2-connexes :
    sous_Graphes = []
    for sommet in Graphe.vertices():
        for voisin in Graphe.neighbors(sommet):
            if (state.temps_decouverte[sommet] < state.temps_decouverte[voisin] and state.parent[voisin] != sommet):
                chaine = {sommet}
                while state.temps_decouverte[voisin] > state.temps_decouverte[sommet]:
                    chaine.add(voisin)
                    voisin = state.parent[voisin]
                sous_Graphe = Graph()
                for n in chaine:
                    for voisin_sous_Graphe in Graphe.neighbors(n):
                        if voisin_sous_Graphe in chaine:
                            sous_Graphe.add_edge(n, voisin_sous_Graphe)
                # Ajouter le sous-graphe à la liste uniquement s'il contient plus d'un sommet
                if len(sous_Graphe.vertices()) > 1 :
                    sous_Graphes.append(sous_Graphe)
                    
    return sous_Graphes


def Est_Decomposable(Graphe):
    #Vérifie si le Graphe est suffisamment grand
    if len(Graphe.vertices()) < 3:
        print("Taille insuffisante pour la décomposition en chaînes")
        return False 
    #Vérifie si le Graphe a des boucles, des arêtes multiples
    arretes = {tuple(sorted(arrete)) for arrete in Graphe.edges(labels=False)}
    if len(arretes) != len(Graphe.edges()) or any(arrete[0] == arrete[1] for arrete in Graphe.edges(labels=False)):
        print("Le Graphe a des arêtes multiples ou des boucles.")
        return False

    return True

def Est_connexe(Graphe, sommet_depart):
    #Vérifie si le Graphe est connexe
    visite = set()
    def dfs(sommet):
        visite.add(sommet)
        for voisin in Graphe.neighbors(sommet):
            if voisin not in visite:
                dfs(voisin)

    dfs(sommet_depart)
    return len(visite) == Graphe.order()

# Exemple Figure 1:--------------------------------------------------------------
Figure1 = Graph([(0, 1), (1, 2), (2, 3), (3, 4), (4, 0), (3, 5), (3, 6), (5, 6)])
#Figure1 = Graph([(1, 2), (2, 3), (3, 4), (4, 1)])
sous_Graphes = Decomposition(Figure1)
print("Exercice 1 Question 1 :")
print("Graphe : Figure 1")
Figure1.show()
if sous_Graphes != False and len(sous_Graphes) !=1:
    print(f"Nombre des composantes 2-connexes du Graphe :{len(sous_Graphes)}")
    for sous_Graphe in sous_Graphes:
        sous_Graphe.show()
elif len(sous_Graphes) ==1:
    print("Le Graphe est 2-connexe")
    

#Exercice 1 Question 2 --------------------------------------------------------------------------------

def DFS_Aretes_Connexes(Graphe, sommet_depart, state, Bridges):
    state.visite.add(sommet_depart)
    state.temps_decouverte[sommet_depart] = state.fin_decouverte[sommet_depart] = state.temps_actuel
    state.temps_actuel += 1
    
    for voisin in Graphe.neighbors(sommet_depart):
        if voisin not in state.visite:
            state.parent[voisin] = sommet_depart
            DFS_Aretes_Connexes(Graphe, voisin, state, Bridges)
            state.fin_decouverte[sommet_depart] = min(state.fin_decouverte[sommet_depart], state.fin_decouverte[voisin])

            if state.fin_decouverte[voisin] > state.temps_decouverte[sommet_depart]:
                # L'arête (sommet_depart, voisin) est une arête pont
                Bridges.append((sommet_depart, voisin))
                
        elif sommet_depart in state.parent and voisin != state.parent[sommet_depart]:
            state.fin_decouverte[sommet_depart] = min(state.fin_decouverte[sommet_depart], state.temps_decouverte[voisin])

def Composantes_connexes(graph):
    visited = set()
    components = []

    def dfs(node, component):
        if node not in visited:
            visited.add(node)
            component.append(node)
            for neighbor in graph.neighbors(node):
                dfs(neighbor, component)

    for node in graph.vertices():
        if node not in visited:
            component = []
            dfs(node, component)
            components.append(component)

    return components
            
def Composantes_Aretes_connexes(Graphe, state):
    # Appel de la fonction DFS_Aretes_Connexes pour trouver les arêtes ponts
    Bridges = []
    DFS_Aretes_Connexes(Graphe, 1, state, Bridges)
    # Supprimer les arêtes ponts du graphe original
    if(len(Bridges) != 0):
        Graphe.delete_edges(Bridges)
        # Trouver les composantes connexes dans le graphe modifié
        composantes = Composantes_connexes(Graphe)
        # Retourner les composantes connexes
        sous_Graphes = [Graphe.subgraph(component) for component in composantes if Graphe.subgraph(component).size() > 1]
    
    else :
        print("le Graphe est 2-aretes connexes ")
        #Pas de composantes 2-arêtes connexes 
        sous_Graphes = []
    # Retourner les sous-graphes
    return sous_Graphes

# Exemple Figure 2:-------------------------------------------------------------
print("Exercice 1 Question 2 :")
Figure2 = Graph([(1, 2), (2, 3), (2, 4), (3, 5), (4, 5), (5, 6), (6, 7), (6, 8),(7, 9), (8, 9), (8, 10), (10, 11), (10, 12), (11, 13), (12, 13),(13, 14), (12, 14)])
#Figure2 = Graph([(1, 2), (2, 3), (1, 3), (3, 4), (3, 7), (4, 10), (10,7), (4,5), (4,6), (5,6), (7,8), (7,9), (8,9), (10,11), (10,12), (11,12)])
state = DFS_State()
print("Graphe : Figure 2")
Figure2.show()
sous_Graphes = Composantes_Aretes_connexes(Figure2, state)
if(len(sous_Graphes)!=0):
    print(f"Nombre des Composantes 2-arêtes connexes du Graphe: {len(sous_Graphes)}")
    for i, sous_Graphe in enumerate(sous_Graphes):
        print(f"Composante {i + 1}:")
        sous_Graphe.show()

#Exercice 2 ------------------------------------------------------------------------------------------------
def est_fortement_connexe(graphe):
    visite = set()

    def dfs(noeud):
        visite.add(noeud)
        for voisin in graphe.neighbors(noeud):
            if voisin not in visite:
                dfs(voisin)

    # Sélectionner un nœud comme point de départ
    start_node = next(iter(graphe.vertices()), None)
    if start_node is not None:
        dfs(start_node)

    # Le graphe est connexe si tous les nœuds ont été visités
    return len(visite) == graphe.order()

def orienter_graphe(graphe, dfs_state):
    if not est_fortement_connexe(graphe):
        print("Le graphe n'est pas connexe.")
        return None

    graphe_oriente = DiGraph()

    def dfs_orient(noeud, parent=None):
        dfs_state.visite.add(noeud)
        dfs_state.parent[noeud] = parent
        for voisin in graphe.neighbors(noeud):
            if voisin not in dfs_state.visite:
                graphe_oriente.add_edge(noeud, voisin)  # Orienter l'arête dans le sens de la visite
                dfs_orient(voisin, noeud)
            elif voisin != parent and dfs_state.parent.get(voisin) is not None:
                # Orienter les arêtes non arborescentes pour former des cycles
                ancetre = dfs_state.parent[voisin]
                while ancetre is not None and ancetre != noeud:
                    ancetre = dfs_state.parent[ancetre]
                if ancetre == noeud:
                    graphe_oriente.add_edge(voisin, noeud)
                else:
                    graphe_oriente.add_edge(noeud, voisin)

    # Sélectionner un nœud comme point de départ
    start_node = next(iter(graphe.vertices()), None)
    if start_node is not None:
        dfs_orient(start_node)

    return graphe_oriente

# Exemple Figure 1----------------------------------------------------------------
print("Exercice 2:")
graphe = Graph([(0, 1), (1, 2), (2, 3), (3, 4), (4, 0), (3, 5), (3, 6), (5, 6)])
dfs_state = DFS_State()
graphe_oriente = orienter_graphe(graphe, dfs_state)

if graphe_oriente:
    print("Graphe Orienté:")
    graphe_oriente.show()
