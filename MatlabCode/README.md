% analyze_cxd_dRR0_fixed_checked.m
% 修复版：加入帧数越界检查，修正 T 索引为 0-based，避免 Invalid T index 错误

%% ---------- 基本设置 ----------
clear; clc;
addpath('C:/bioformats/bio-formats-matlab-master/bio-formats-matlab-master/src');
javaaddpath('C:/bioformats/bioformats_package.jar');

filePath = [''];

roiRadius = 20;
bgRegion = [1, 1, 30, 30];

stimFrame = input('>> 请输入制动开始帧号：');
endStimFrame = input('>> 请输入制动结束帧号：');

%% ---------- 读取图像 ----------
r = bfGetReader(filePath);
r.setSeries(0);
sizeX = r.getSizeX(); sizeY = r.getSizeY();
sizeC = r.getSizeC(); sizeT = r.getSizeT();

fprintf('📢 当前图像总帧数（T维）为：%d\n', sizeT);

if sizeC < 2
    error('>> 通道数量小于2，无法分离 GFP/mCherry');
end
if stimFrame < 1 || stimFrame > sizeT
    error('❌ 输入的刺激开始帧超出范围，合法范围是 1 ~ %d', sizeT);
end
if endStimFrame < stimFrame || endStimFrame > sizeT
    error('❌ 输入的刺激结束帧不合法，应 ≥ 开始帧且 ≤ %d', sizeT);
end

baselineFrames = (stimFrame-11):(stimFrame-1);
stimFrames = stimFrame : min(stimFrame + 468, endStimFrame);
postStimFrames = (endStimFrame+1) : (endStimFrame+60);
allFrames = unique([baselineFrames, stimFrames, postStimFrames]);
allFrames = allFrames(allFrames >= 1 & allFrames <= sizeT);
if isempty(allFrames)
    error('❌ 有效帧为空，请检查 stimFrame 和 endStimFrame 设置');
end
validFrames = allFrames;
nFrames = length(validFrames);

[~, nameOnly, ~] = fileparts(filePath);
outputDir = fullfile('results', nameOnly);
if ~exist(outputDir, 'dir')
    mkdir(outputDir);
end

%% ---------- 首帧手动选点 ----------
z = 0; c_mCherry = 1;
index = r.getIndex(z, c_mCherry, stimFrame - 12);
img_mCherry = bfGetPlane(r, index+1);
figure, imshow(img_mCherry, []), title('选择神经元位置（stimFrame前第14帧）');
[x0, y0] = ginput(1);
close;

roiBox = [round(x0)-roiRadius, round(y0)-roiRadius, 2*roiRadius, 2*roiRadius];
roiBox(1:2) = max(roiBox(1:2), 1);
roiBox(3) = min(roiBox(3), sizeX - roiBox(1));
roiBox(4) = min(roiBox(4), sizeY - roiBox(2));
template_img = double(img_mCherry(roiBox(2):roiBox(2)+roiBox(4), roiBox(1):roiBox(1)+roiBox(3)));
[X, Y] = meshgrid(1:sizeX, 1:sizeY);

R_values = zeros(1, nFrames);
xy_traj = zeros(nFrames, 2);
Fgfp_raw_all = zeros(nFrames, 1);
Fchy_raw_all = zeros(nFrames, 1);
Fgfp_bg_all = zeros(nFrames, 1);
Fchy_bg_all = zeros(nFrames, 1);
Fgfp_all = zeros(nFrames, 1);
Fchy_all = zeros(nFrames, 1);

video = VideoWriter(fullfile(outputDir, [nameOnly '_tracking.avi']));
open(video);

%% ---------- 主循环 ----------
figure('visible','off');
R0 = NaN;  % 初始化 R0
for i = 1:nFrames
    t = validFrames(i);

    idxGFP = r.getIndex(z, 0, t - 1);
    idxChy = r.getIndex(z, 1, t - 1);
    imgGFP = double(bfGetPlane(r, idxGFP+1));
    imgChy = double(bfGetPlane(r, idxChy+1));

    if i == 1
        x = x0; y = y0;
    else
        searchRadius =200;
        xCenter = round(x0); yCenter = round(y0);
        x1 = max(1, xCenter - searchRadius);
        x2 = min(sizeX, xCenter + searchRadius);
        y1 = max(1, yCenter - searchRadius);
        y2 = min(sizeY, yCenter + searchRadius);

        subImg = imgChy(y1:y2, x1:x2);
        c = normxcorr2(template_img, subImg);
        [~, imax] = max(c(:));
        [ypeak, xpeak] = ind2sub(size(c), imax);
        y = y1 + ypeak - size(template_img,1);
        x = x1 + xpeak - size(template_img,2);
    end

    mask = sqrt((X - x).^2 + (Y - y).^2) <= roiRadius;
    xy_traj(i, :) = [x, y];

    bgGFP = mean(imgGFP(bgRegion(2):(bgRegion(2)+bgRegion(4)-1), bgRegion(1):(bgRegion(1)+bgRegion(3)-1)), 'all');
    bgChy = mean(imgChy(bgRegion(2):(bgRegion(2)+bgRegion(4)-1), bgRegion(1):(bgRegion(1)+bgRegion(3)-1)), 'all');

    Fgfp_raw = mean(imgGFP(mask), 'all');
    Fchy_raw = mean(imgChy(mask), 'all');
    Fgfp = Fgfp_raw - bgGFP;
    Fchy = Fchy_raw - bgChy;

    Fgfp_raw_all(i) = Fgfp_raw;
    Fchy_raw_all(i) = Fchy_raw;
    Fgfp_bg_all(i) = bgGFP;
    Fchy_bg_all(i) = bgChy;
    Fgfp_all(i) = Fgfp;
    Fchy_all(i) = Fchy;
    R_values(i) = Fgfp / Fchy;

    % ✅ 在满足所有 baseline frame 有数据后计算 R₀（刺激帧前10帧的 R 平均）
    if isnan(R0)
        fixedBaselineFrames = (stimFrame - 10):(stimFrame - 1);
        [~, baselineIdx] = ismember(fixedBaselineFrames, validFrames(1:i));
        baselineIdx = baselineIdx(baselineIdx > 0);
        if numel(baselineIdx) == 10
            R0 = mean(R_values(baselineIdx));
        end
    end

    clf;
    subplot(2,1,1);
    imshow(mat2gray(imgChy)); hold on;
    viscircles([x y], roiRadius, 'Color', 'r');
    text(10, 20, sprintf('Frame %d', t), 'Color', 'y', 'FontSize', 14);
    title('mCherry with Neuron ROI');

    subplot(2,1,2);
    if ~isnan(R0)
        dRtemp = (R_values(1:i) - R0) / R0;
        plot(validFrames(1:i), dRtemp, 'b'); hold on;
        xline(stimFrame, '--r');
        xlim([validFrames(1), validFrames(end)]);
        ylim([-1 2]);
        ylabel('\DeltaR/R_0');
        title('\DeltaR/R_0 trace');
    end

    frame = getframe(gcf);
    writeVideo(video, frame);
end
close(video);

%% ---------- 最终计算 ΔR/R0 ----------
dR_R0_raw = (R_values - R0) / R0;
dR_R0_smooth = smoothdata(dR_R0_raw, 'movmean', 5);

%% ---------- 输出 ----------
T = table((validFrames'), R_values', repmat(R0, nFrames, 1), dR_R0_raw', dR_R0_smooth', ...
          Fgfp_raw_all, Fgfp_bg_all, Fgfp_all, ...
          Fchy_raw_all, Fchy_bg_all, Fchy_all, ...
          'VariableNames', {'Frame', 'R', 'R0', 'dR_R0_raw', 'dR_R0_smooth', ...
                            'Fgfp_raw', 'Fgfp_bg', 'Fgfp', 'Fchy_raw', 'Fchy_bg', 'Fchy'});
T(1,:) = [];
dR_R0_smooth(1) = [];
dR_R0_raw(1) = [];
R_values(1) = [];
writetable(T, fullfile(outputDir, [nameOnly '_combined_results.xlsx']));

figure;
plot(T.Frame, dR_R0_smooth, 'b', 'LineWidth', 1.5);
xline(stimFrame, '--r', 'stim');
xline(endStimFrame, '--m', 'stop', 'LabelOrientation','horizontal','LabelVerticalAlignment','middle','Color','m');
xlabel('Frame'); ylabel('\DeltaR/R_0');
title('\DeltaR/R_0 Trace');
grid on;
saveas(gcf, fullfile(outputDir, [nameOnly '_dRR0_plot.png']));
