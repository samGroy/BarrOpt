"# BarrOpt" 
%Barrier Optimization (BarrOpt.m) cookbook
%Sam Roy, UMaine Mitchell Center, 3/13/20
%Place this older in your C:\ drive
%This script contains all commands that are required to prepare and run a
%simulation in BarrOpt.m, a Matlab program that accesses Matlab's
%multi-objective optimization functions to identify pareto efficient
%decisioin scenarios. These decision scenarios are presented as a series, each with
%lists of decisions for multiple dams and culverts. The basic code is:
%0 = keep dam and culvert as they are,
%1 = remove dam, or replace culvert with upgraded safety/eco standards

%first, define the objective variables you want to optimize for (vars) and
%the study region. For a 3D trade-off assesment we use river herring habitat (herring), road safety
%risk (risk), and economic cost of decisions (cost). There are other
%variables available, but these will do for now. For study region, 'All'
%includes the entire region in Maine.
fprintf('defining variables and study region\n')
vars = {'herring','risk','cost'};
StudyRegion = 'All';

%Second, we need to retrieve and preprocess the culvert/dam data.
%Specifically we need to access an index file (BarrIndex) for dams and culverts in
%order to attribute variable values to the correct feature, but the index
%is also important for ordering the dams and culverts based on their
%position in river/stream networks. We also need to retrieve the variable values
%for each culvert/dam (v).
%But first, we need to direct the program to the directory where these data
%are located. This WILL need to be altered to find where these files are on
%your computer:
fprintf('Preparing data for GA\n')
data_directory = 'C:\BarrOpt_software';
[BarrIndex, v] = BarrOpt_prep(StudyRegion, data_directory, 'crh1',vars);
%The 'crh1' code just means that we're only selecting existing dams/culverts

%Third, we need to prescribe an initial 'population' for the genetic
%algorithm to start with. The genetic algorithm is designed based on a very
%general interpretation of how evolution works. It will come up with
%different 'populations', where each population offers a different decision
%for each dam/culvert in the study region. The model iterates through
%several populations and keeps members of the population (representing
%decision scenarios) that are more 'fit' than others, and therefore offer
%more efficiency when trading off between habitat, road safety, and cost.
%But, you can help the algorithm out by giving it starting populations, and
%the best way to do this (I have found by lots of trial and error) is to
%give it 'extreme' cases to start with, and it will eventually fill in the
%gradient between these. The extreme cases I use are: 'keep all dams and
%culverts as they are' (all decisions = 0) and 'remove all dams and replace
%all culverts' (all decisions = 1). Here is how to define these
%populations:
fprintf('initializing start population\n')
if size(BarrIndex,2)==2
    total_number_barrs = length(BarrIndex(BarrIndex(:,1)<1e9,1));
else
    total_number_barrs = length(BarrIndex(BarrIndex(:,1)<1e9,1))-1;
end
xx=zeros(2,total_number_barrs); %this is 'remove/replace all'
xx(1,:) = 1; %this is 'keep all'

%Fourth, we want to prescribe a population size (pop) for the model. Larger
%populations are better at finding efficient solutions, but if the
%population is too big it will take to long to generate a solution/crash.
%So, it's dependent on your processor speed, ram, etc. so we can fiddle
%with this number. 2000 tends to be ideal, but we can start smaller:
pop = 100;

%Fourth, we can now run the genetic algorithm (GA)! We do so using a 'shell'
%program that feeds our data into the GA and calculates the 'fitness' of
%each population, or in other words, the relative efficiency of each
%scenario. %There are a lot of outputs from this (variables left of the
%equal sign) but we really only care about x and f.
%FYI, this simulation could take up to an hour, even more for a larger
%population.
fprintf('run 1...\n')
[x, f, xu, fu, exitflag, output, population, score] = BarrOpt(BarrIndex,pop,xx,v);
fprintf('complete\n')

%Fifth, I find that rerunning GA several times leads to more efficient results.
%Sometimes this can lead to sudden improvements in efficiency. I run
%BarrOpt again using the output population (x) as input. If run times
%are long, take some reruns out.
%Keep this commented out until confident about GA use
%fprintf('run 2...\n')
%[x, f, xu, fu, exitflag, output, population, score] = BarrOpt(BarrIndex,pop,x,v);
%fprintf('complete\n')
%fprintf('run 3...\n')
%[x, f, xu, fu, exitflag, output, population, score] = BarrOpt(BarrIndex,pop,x,v);
%fprintf('complete\n')

%Sixth, visualize the results. Because we have three objective variables,
%we will genreate a 3D surface plot of points defining a production
%possibility frontier 'surface'
fprintf('plotting frontier\n')
plot3(f(:,1),f(:,2),f(:,3),'.k')

%now save your results, using an easily identifiable name and date of
%simulation. It's very useful to specify the objective variables you have
%used and the population:
fprintf('saving run\n')
t=datestr(datetime('now'),'dd-mmm-yyyy_HH-MM-SS');
save(sprintf('%s\\BarrOut\\Scenarios_%s_herring_risk_cost_pop100_%s.mat',data_directory,StudyRegion,t))
